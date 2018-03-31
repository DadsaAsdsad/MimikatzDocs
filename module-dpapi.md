# A basic introduction

## A `blob`

* contains: encrypted raw data, secret, by example Vault, Credential, CAPI/CNG Private Key, Chrome password, WiFi/WWAN key, ...
* is used to: _what you want!_, this is the final data
* is protected by: a `masterkey` and optionally `entropy` data **AND/OR** aditionnal `password`
* is linked to: a `masterkey`

## A `masterkey`

* contains: multiple versions of the encrypted raw key
* is used to: decrypt `blob`
* is protected by: a key that depends on the situation
  * non-domain context: SID **AND** (user password SHA1 hash **OR** previous password SHA1 hash (by knowledge or from `CREDHIST`))
  * domain context:
    * SID **AND** (user password NTLM hash **OR** previous password NTLM hash (by knowledge))
    * domain backup key (`RPC` or RSA private key)
  * local computer: `DPAPI_SYSTEM` secret (`COMPUTER` or `USER` part)
* is linked to: a `credhist` entry

## A `credhist`

_Only useful in non-domain context_
* contains: previous encrypted credentials of the user (SHA1 & NTLM)
* is used to: decrypt `masterkey`
* is protected by: the most recent user password SHA1 hash used by the user on the system
  * each entry is protected by the previous key, etc.

## Remarks

You can find the location of these files: https://1drv.ms/x/s!AlQCT5PF61KjmCAhhYO0flOcZE4e  
When the user is a protected user, it's **NOT** the NTLM hash of the password, but another derived hash from it.

# Commands:

> Commands: [blob](#blob), [protect](#protect), [masterkey](#masterkey), [credhist](#credhist), [cache](#cache), [capi](#capi), [cng](#cng), [cred](#cred), [vault](#vault), [wifi](#wifi), [wwan](#wwan), [chrome](#chrome)


## blob
**Arguments:**
* `/in` - 
* `/out` - _optional_ - 

**Generic arguments (all are optional) :**
* `/unprotect` -  
* `/masterkey` - 
* `/password` - 
* `/entropy` - 
* `/prompt` - 
* `/machine` - 

## protect
**Arguments:**
* `/data` - 
* `/description` - _optional_ - 
* `/entropy` - _optional_ - 
* `/machine` - _optional_ - 
* `/system` - _optional_ - 
* `/prompt` - _optional_ - 
* `/out` - _optional_ - 

## masterkey
**Arguments:**
* `/in` - 
* `/sid` - _optional_ - 
* `/hash` - _optional_ - 
* `/system` - _optional_ - 
* `/password` - _optional_ - 
* `/protected` - _optional_ - 
* `/pvk` - _optional_ - 
* `/rpc` - _optional_ - 
* `/dc` - _optional_ - 
* `/domain` - _optional_ - 

## credhist
**Arguments:**
* `/in` - 
* `/sid` - _optional_ - 
* `/password` - _optional_ - 
* `/hash` - _optional_ - 

## cache
Display the credential cache of the DPAPI module

```
mimikatz # dpapi::cache

CREDENTIALS cache
=================
SID:S-1-5-21-1982681256-1210654043-1600862990-1000;GUID:{a62828b1-b384-408c-8c78-90607b9c6e53};MD4:cc36cf7a8514893efccd332446158b1a;SHA1:a299912f3dc7cf0023aef8e4361abfc03e9a8c30;
SID:S-1-5-21-1982681256-1210654043-1600862990-1000;GUID:{4a108cb4-6080-4ac3-a76a-bb738c98b212};MD4:31d6cfe0d16ae931b73c59d7e0c089c0;SHA1:da39a3ee5e6b4b0d3255bfef95601890afd80709;

MASTERKEYS cache
================
GUID:{6fc9a9bb-e99f-4030-8fdc-fcce59373509};KeyHash:91e886dc63db8d343e9c3d95b849dd4ee2517362
GUID:{8d2c893f-73a7-4874-bf35-af4876ef68d1};KeyHash:a4cd97b9250543d32ef088be396b9471922cffc3

DOMAINKEYS cache
================
GUID:{945680db-66ad-49bb-a44d-26d2a3480578};TYPE:RSA
GUID:{a0d107d2-eb19-4460-98a8-142e836a19e3};TYPE:LEGACY
```

## capi
## cng
## cred
## vault
## wifi
## wwan
## chrome