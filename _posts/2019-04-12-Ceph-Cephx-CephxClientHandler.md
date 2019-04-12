---
layout: post
title:  "Cephx - CephxClientHandler 코드 분석"
date:   2019-04-12 15:01:59
author: Hanjoo Cho
categories: Cloud
tags:    Cloud Ceph
cover:  "/assets/instacode.png"
---
# Ceph 코드 분석 - cephx

## - 경로 : ceph/src/auth/cephx/CephxClientHandler.h

```c++
class CephxClientHandler : public AuthClientHandler {
  bool starting;

  /* envelope protocol parameters */
  uint64_t server_challenge;

  CephXTicketManager tickets;
  CephXTicketHandler* ticket_handler;

  RotatingKeyRing* rotating_secrets;
  KeyRing *keyring;

public:
  CephxClientHandler(CephContext *cct_,
		     RotatingKeyRing *rsecrets)
    : AuthClientHandler(cct_),
      starting(false),
      server_challenge(0),
      tickets(cct_),
      ticket_handler(NULL),
      rotating_secrets(rsecrets),
      keyring(rsecrets->get_keyring())
  {
    reset();
  }

  void reset() override;
  void prepare_build_request() override;
  int build_request(bufferlist& bl) const override;
  int handle_response(int ret, bufferlist::const_iterator& iter,
		      CryptoKey *session_key,
		      std::string *connection_secret) override;
  bool build_rotating_request(bufferlist& bl) const override;

  int get_protocol() const override { return CEPH_AUTH_CEPHX; }

  AuthAuthorizer *build_authorizer(uint32_t service_id) const override;

  bool need_tickets() override;

  void set_global_id(uint64_t id) override {
    global_id = id;
    tickets.global_id = id;
  }
private:
  void validate_tickets() override;
  bool _need_tickets() const;
};
```



## - 경로 : ceph/src/auth/cephx/CephxClientHandler.cc

**> build_request(bufferlist& bl) const** 

### 1. request to create a user

클라이언트는 사용자 이름을 모니터에 전달

```c++
int CephxClientHandler::build_request(bufferlist& bl) const
{
  ldout(cct, 10) << "build_request" << dendl;

  if (need & CEPH_ENTITY_TYPE_AUTH) {
    /* authenticate */
    CephXRequestHeader header;
    header.request_type = CEPHX_GET_AUTH_SESSION_KEY;
    encode(header, bl);

    CryptoKey secret;
    const bool got = keyring->get_secret(cct->_conf->name, secret);
    if (!got) {
      ldout(cct, 20) << "no secret found for entity: " << cct->_conf->name << dendl;
      return -ENOENT;
    }

    // is the key OK?
    if (!secret.get_secret().length()) {
      ldout(cct, 20) << "secret for entity " << cct->_conf->name << " is invalid" << dendl;
      return -EINVAL;
    }
```

### - 경로 : ceph/src/auth/KeyRing.h

KeyRing 정의 - get_secret

```c++
class KeyRing : public KeyStore {
  std::map<EntityName, EntityAuth> keys;

  int set_modifier(const char *type, const char *val, EntityName& name, std::map<std::string, ceph::buffer::list>& caps);
public:
  void decode_plaintext(ceph::buffer::list::const_iterator& bl);
  /* Create a KeyRing from a Ceph context.
   * We will use the configuration stored inside the context. */
  int from_ceph_context(CephContext *cct);

  /* Create an empty KeyRing */
  static KeyRing *create_empty();

  std::map<EntityName, EntityAuth>& get_keys() { return keys; }  // yuck


...
  bool get_secret(const EntityName& name, CryptoKey& secret) const override {
    std::map<EntityName, EntityAuth>::const_iterator k = keys.find(name);
    if (k == keys.end())
      return false;
    secret = k->second.key;
    return true;
  }
 
...
```



### 2. transit key

세션 키를 검색하기 위해 공유 비밀 키로 페이로드를 암호 해독

~~~c++
		CephXAuthenticate req;
    req.client_challenge = ceph::util::generate_random_number<uint64_t>();
    std::string error;
    cephx_calc_client_server_challenge(cct, secret, server_challenge,
				       req.client_challenge, &req.key, error);
    if (!error.empty()) {
      ldout(cct, 20) << "cephx_calc_client_server_challenge error: " << error << dendl;
      return -EIO;
    }
~~~



### - 경로 : ceph/src/auth/cephx/CephxProtocol.cc

```c++
void cephx_calc_client_server_challenge(CephContext *cct, CryptoKey& secret, uint64_t server_challenge, 
		  uint64_t client_challenge, uint64_t *key, std::string &error)
{
  CephXChallengeBlob b;
  b.server_challenge = server_challenge;
  b.client_challenge = client_challenge;

  bufferlist enc;
  if (encode_encrypt(cct, b, secret, enc, error))
    return;

  uint64_t k = 0;
  const uint64_t *p = (const uint64_t *)enc.c_str();
  for (int pos = 0; pos + sizeof(k) <= enc.length(); pos+=sizeof(k), p++)
    k ^= mswab(*p);
  *key = k;
}
```



### 3. req. ticket

세션 키로 서명 한 사용자를 대신하여 티켓을 요청

```c++
if (_need_tickets()) {
    /* get service tickets */
    ldout(cct, 10) << "get service keys: want=" << want << " need=" << need << " have=" << have << dendl;

    CephXRequestHeader header;
    header.request_type = CEPHX_GET_PRINCIPAL_SESSION_KEY;
    encode(header, bl);

    CephXAuthorizer *authorizer = ticket_handler->build_authorizer(global_id);
    if (!authorizer)
      return -EINVAL;
    bl.claim_append(authorizer->bl);
    delete authorizer;

    CephXServiceTicketRequest req;
    req.keys = need;
    encode(req, bl);
  }

  return 0;
}
```

