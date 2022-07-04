### AnonCreds Issuance Data Flow

The issuance of an anonymous [[ref:credential]] requires several steps and involves the roles [[ref:issuer]], [[ref:holder]] as well as the [[ref: Verifiable Data Registry]] (see diagramm below).

```mermaid
sequenceDiagram
    autonumber
    participant L as Verifiable<br>Data Registry
    participant I as Issuer
    participant H as Holder  

  I ->> I: Create Credential Offer
  I ->> H: Send Credential Offer
  H ->> H: Process Credential Offer
  opt
    H ->> L: Request SCHEMA
    L ->> H: Return SCHEMA
  end
  H ->> L: Request CRED_DEF
  L ->> H: Return CRED_DEF
  H ->> H: Create Credential Request
  H ->> I: Send Credential Request
  I ->> I: Process Credential Request
  I ->> I: Issue Credential
  I ->> H: Send Credential
  H ->> H: Store Credential


  rect rgb(191, 223, 255)
    Note left of H: 💡The "Verifier" and "Schema Publisher" roles are<br>omitted in this diagram, since they is not required<br>for the credential issuance data flow.
  end
```

The [[ref:issuer]] prepares a [[ref:Credential Offer]] for the [[ref:holder]] (step 1). A [[ref:Credential Offer]] includes information about what kind of [[ref:credential]] (based on which [[ref: CRED_DEF]]) the [[ref:issuer]] is intending to issue to the [[ref:holder]]. The [[ref:issuer]] sends the [[ref:Credential Offer]] to the [[ref:holder]] (step 2), who then evaluates the incoming offer (step 3) and subsequently fetches required data (the [[ref:CRED_DEF]]) from the [[ref: Verifiable Data Registry]] (step 4-7).

Based on the [[ref:CRED_DEF]] received from the [[ref:Verfiable Data Registry]], the [[ref:holder]] prepares a [[ref:Credential Request]] (step 8). A [[ref: Credential Request]] is a formal request from a [[ref:holder]] to an [[ref:issuer]] to get a [[ref:credential]] based on the given [[ref:CRED_DEF]] issued to the [[ref:holder]]. The [[ref:holder]] sends the [[ref: Credential Request]] to the [[ref:issuer]] (step 9), who then evaluates the incoming request (step 10).

The [[ref:issuer]] can decide whether to accept the received [[ref: Credential Request]] and issues the [[ref:credential]] (step 11) in the case of request acceptance. The [[ref:issuer]] sends the credential to the [[ref:holder]] (step 12), who then can store the received [[ref:credential]] in his wallet (step 13).


#### Credential Offer

Before issuing a credential to the [[ref:holder]], the [[ref:issuer]] has to send a [[ref:Credential Offer]] to the potential [[ref:holder]] (step 1 and 2). A [[ref:Credential Offer]] contains information about the credential the [[ref:issuer]] intends to issue and send to the [[ref:holder]]. For creating a [[ref:Credential Offer]], the [[ref:issuer]] is required to fetch the [[ref:CRED_DEF]] as well as its correctness proof from the [[ref: Verifiable Data Registry]]. The [[ref:issuer]] also prepares a [[ref:nonce]] which will be embedded within the [[ref:Credential Offer]] in order to prevent replay attacks and authenticate between protocol steps.

The resulting JSON for a created [[ref:Credential Offer]] is shown here:

```json
{
    "schema_id": string,
    "cred_def_id": string,
    // Fields below can depend on Cred Def type
    "nonce": string,
    "key_correctness_proof" : <key_correctness_proof>
}
```
* `schema_id`: The ID of the [[ref:SCHEMA]] on which the [[ref:CRED_DEF]] for the offered [[ref:Credential]] is based.
* `cred_def_id`: The ID of the [[ref:CRED_DEF]] on which the [[ref:Credential]] to be issued will be based.
* `nonce`: Used for preventing replay attacks and authentication between protocol steps. Has to be reused within a [[ref:Credential Request]] by the [[ref:holder]].
* `key_correctness_proof`: TODO.

The [[ref:issuer]] sends the [[ref:Credential Offer]] to the [[ref:holder]] (step 2), who then can process the [[ref: Credential Offer]] (step 3). In order to figure out, which kind of credential (which [[ref:CRED_DEF]] and attributes) are offered to the [[ref:holder]], the [[ref:holder]] needs to fetch the underlying [[ref:SCHEMA]] from the [[ref:Verifiable Data Registry]] by using the ```schema_id``` as provided in the received [[ref:Credential Offer]], since the referenced [[ref:SCHEMA]] contains all attribute names of the offered credential (step 4 + 5). 

In case the credential respectively its attributes is of interest for the [[ref:holder]], the [[ref:holder]] can reply to the issuer`s [[ref:Credential Offer]] with a [[ref:Credential Request]] in order to ask the issuer for issuance of the offered credential and its attributes.

:::todo
 - Add info to key_correctness_proof
:::

#### Credential Request

A [[ref:Credential Request]] is a formal request from a [[ref:holder]] to an [[ref:issuer]] to get a [[ref:credential]] based on a concrete [[ref:CRED_DEF]] issued by the [[ref:issuer]] to the [[ref:holder]]. 

In order to be able as a [[ref:holder]] to express within a [[ref:Credential Request]] to the [[ref:issuer]] which kind of credential the [[ref:issuer]] shall issue to the [[ref:holder]], the [[ref:holder]] requires the [[ref:CRED_DEF]] from the [[ref:Verifiable Data Registry]] if not already available in local storage (step 6 + 7). The [[ref:Credential Request]] has to reference the same [[ref:CRED_DEF]] and [[ref:nonce]] as given in the preceding [[ref:Credential Offer]]. Besides the [[ref:CRED_DEF]], the [[ref:holder]] also requires his [[ref:link secret]] in a blinded form, as well as the corresponding [[ref: Correctness Proof]] of his [[ref:link secret]]. The [[ref: holder]] has now all relevant data for creating the [[ref:Credential Request]] (step 8).

::: todo
- Add here: How does the link secret get blinded? How does the cryptography work? How does it work with correctness proof? 
:::

The resulting JSON for a created [[ref:Credential Request]] is shown here:

```json
{
  "prover_did" : string,
  "cred_def_id" : string,
  // Fields below can depend on Cred Def type
  "blinded_ms" : string,
  "blinded_ms_correctness_proof" : string,
  "nonce": string
}
```

:::todo
is the ```prover_did``` the peer DID of the holder?
:::


* `prover_did`: The [[ref:DID]] of the [[ref:holder]].
* `cred_def_id`: The ID of the [[ref:CRED_DEF]] on which the [[ref:Credential]] to be issued shall be based.
* `blinded_ms`: The [[ref:link secret]] in its blinded form.
* `blinded_ms_correctness_proof`: The [[ref: Correctness Proof]] of the blinded [[ref:link secret]].
* `nonce`: Used for preventing replay attacks and authentication between protocol steps. Reused from the initially received [[ref:Credential Offer]].

The [[ref:issuer]] sends the [[ref:Credential Request]] to the [[ref:issuer]] (step 9), who then can reply to the [[ref:holder]] by sending an issued credential.


#### Issue Credential without revocation

After the [[ref:issuer]] received the [[ref:Credential Request]] from the [[ref:holder]], the [[ref:issuer]] processes the [[ref:Credential Request]] and decides whether to issue the credential as requested in the [[ref:Credential Request]] to the [[ref:holder]]. 

In case the [[ref:issuer]] decides to issue the requested credential to the [[ref:holder]], the following steps have to be executed by the [[ref:issuer]]:

:::todo
- check nonce?
- check link secret and correctness proof?
:::

1. The [[ref:issuer]] has to fetch the [[ref:CRED_DEF]] for the `cred_def_id` given in the received [[ref:Credential Request]] either from the ledger or local storage (if already available).
2. Every raw attribute value for each attribute in the fetched [[ref:CRED_DEF]] (respectively its [[ref:Schema]]), which the [[ref:issuer]] intends to issue to the [[ref:holder]], needs to be set.
4. Every raw attribute value, which cannot successfully be parsed into an integer (e.g. "Alice"), must be encoded as integer. The same rule applies to the blinded [[ref:link secret]].
5. Every raw attribute value, which can successfully be parsed into an integer (e.g. "2015"), shall not be encoded explicitely. In this case it is required to use the raw (integer) value also as the encoded one. The intermediate result of raw and encoded credential attribute values is as follows (cred_values_json):

```json
{
    "first_name": {"raw": "Alice", "encoded": "1139481716457488690172217916278103335"},
    "last_name": {"raw": "Garcia", "encoded": "5321642780241790123587902456789123452"},
    "degree": {"raw": "Bachelor of Science, Marketing", "encoded": "12434523576212321"},
    "status": {"raw": "graduated", "encoded": "2213454313412354"},
    "ssn": {"raw": "123-45-6789", "encoded": "3124141231422543541"},
    "year": {"raw": "2015", "encoded": "2015"},
    "average": {"raw": "5", "encoded": "5"},
}
```

6. The [[ref:issuer]] has to fetch the [[ref:holder]]`s blinded [[ref:link secret]] from the received [[ref:Credential Request]]. The blinded [[ref:link secret]] is available in the received [[ref:Credential Request]] at ```blinded_ms```. 
7. The [[ref:issuer]] has to sign each attribute value and the the blinded [[ref:link secret]] by using the corresponding private key for each attribute as defined in the private part of the [[ref:CRED_DEF]] earlier.

:::todo
- check how exactly the signing happens for the whole credential
:::

The [[ref:issuer]] has to transmit the whole credential data to the [[ref:holder]] as follows (the shown data is based on a credential that has revocation not activated):


```json
{
    "schema_id": string,
    "cred_def_id": string,
    "rev_reg_id": null,
    "values": {
        "first_name": {
            "raw": "Alice",
            "encoded": "1139481716457488690172217916278103335"
        },
        "last_name": {
            "raw": "Garcia",
            "encoded": "5321642780241790123587902456789123452"
        },
        "degree": {
            "raw": "Bachelor of Science, Marketing",
            "encoded": "12434523576212321"
        },
        "status": {
            "raw": "graduated",
            "encoded": "2213454313412354"
        },
        "ssn": {
            "raw": "123-45-6789",
            "encoded": "3124141231422543541"
        },
        "year": {
            "raw": "2015",
            "encoded": "2015"
        },
        "average": {
            "raw": "5",
            "encoded": "5"
        }
    },
    "signature": {
        "p_credential": {
            "m_2": "99219524012997799443220800218760023447537107640621419137185629243278403921312",
            "a": "548556525746779881166502363060885...94013062295153997068252",
            "e": "25934472305506205990702549148069757193...84639129199",
            "v": "97742322561796582616103087460253...25643543159082080893049915977209167597"
        },
        "r_credential": null
    },
    "signature_correctness_proof": {
        "se": "898650024692810554511924969312...143339518371824496555067302935",
        "c": "93582993140981799598406702841334282100000866001274710165299804498679784215598"
    },
    "rev_reg": null,
    "witness": null
}
```

:::todo
- what is the naming scheme for the CL signatures in p_credential? Since the shown JSON is the result of two mixed examples, the signatures for more than the three presented attributes are missung. m_2 is the link secret...
:::

* `schema_id`: The ID of the [[ref:SCHEMA]] on which the [[ref:CRED_DEF]] for the offered [[ref:Credential]] is based.
* `cred_def_id`: The ID of the [[ref:CRED_DEF]] on which the [[ref:Credential]] issued is based.
* `values`: The raw and encoded credential attribute values as JSON (cred_values_json).
* `signature`: The signatures of the separately signed attributes
  * `p_credential`: The primary credential that includes all credential values
* `signature_correctness_proof`: The signature correctness proof of the signature for the whole credential data.

:::todo
- Add description for remaining keys of json shown above
- What kind of encoding algorithm for strings is used? Seems like this is not defined explicitely (https://jira.hyperledger.org/browse/IS-786)
- Go deeper into signing with CL?
- Encoding the raw blinded link secret value and using it as encoded one is correct?
- consider revocation data in case of revocation
:::

After the [[ref:issuer]] sent the credential data to the [[ref:holder]] (step 12), the [[ref:holder]] can accept and store the credential within his wallet (step 13). The credential issuance flow is completed at this point.

#### Issue Credential with revocation
If revocation has been enabled in the credential definition further steps are necessary during the issuance flow of a credential with revocation. This means that in addition to the described flow in [issuer setup](data_flow_issuance.md#issue-credential-without-revocation) section a seperate revocation credential has to be created as well as related information for revocation verification.

1. ...
2. ...
3. ...
4. ...
5. ...
6. ...
7. The [[ref:issuer]] has to add the ```accumulator``` value (for which an index from the related ```tails file``` was taken) and the ```wittness value``` which both need to be published. In the existing Hyperledger Indy implementation of AnonCreds those values are published on the ledger. [TODO: add ref to revocation]
8. The [[ref:issuer]] has to create a non revocation credential signature ```r_credential``` 
9. The [[ref:issuer]] must publish a revocation registry entry on the Ledger [TODO: add ref to revocation]
10. The [[ref:issuer]] has to sign each attribute value and the the blinded [[ref:link secret]] by using the corresponding private key for each attribute as defined in the private part of the [[ref:CRED_DEF]] earlier.

The [[ref:issuer]] has to transmit the whole credential data to the [[ref:holder]] as follows (the shown data is based on a credential that has revocation activated):
```json
{
    "schema_id": string,
    "cred_def_id": string,
    "rev_reg_id": string,
    "values": {...},
    "signature": {
        "p_credential": {...},
        "r_credential": {
            "sigma": "1 15518248A4B5ED5C71D60FFF3D91E60221A8D22FD7A84B07CA088627552476FD 1 18ED7014061EFBC67584F8885EF2185F1957EF217A3939724D92871BF7780433 2 095E45DDF417D05FB10933FFC63D474548B7FFFF7888802F07FFFFFF7D07A8A8",
            "c": "142B1A89E4657EFD888548A840888B92F42DC8F2F88F55FC19E72B6F4F316AA5",
            "vr_prime_prime": "1076AA813251B83AB2606D995646E2CC3B754CCEF63FD8379973BA69B3D02900",
            "witness_signature": {
                "sigma_i": "1 18A415A1CFB0B4CFAB5E2FB2DAD539F7B321002935CB78B45B0F0F1085E881FE 1 0CB32E3790CA7F59243F5636A02A10E14F3FBE5374A82E5DAC3FDF56FD8DE852 1 00651A6FA54977BBD1ACCC09616AAF61CEAB10E21D5436EF80A63E1FA7A1C0F9 1 0B9A4D51A09B17AE2431DCE7E1DAA74E1DBBD08161543A4A2BE2BDD59A278A7B 2 095E45DDF417D05FB10933FFC63D474548B7FFFF7888802F07FFFFFF7D07A8A8 1 0000000000000000000000000000000000000000000000000000000000000000",
                "u_i": "1 05421512170D3E432A85A5F59C9FFE995A64C533E1F5DBFF3526F4717D998897 1 251E8140D1AE342112EE237C2257D554AD0A2D97588D42A3EB7A2C20B719A368 1 1F0059E8FC2051D4388CF16F2F5452639AD521C70E499B0E64A93932EC1FD897 1 173B52089498AF197B8F5CF01F9A380A61210A31E9EDDDB444BB1B88C2745D30 2 095E45DDF417D05FB10933FFC63D474548B7FFFF7888802F07FFFFFF7D07A8A8 1 0000000000000000000000000000000000000000000000000000000000000000",
                "g_i": "1 0FC61615A31975C74D052AC7E246A556FC6D4696C58636B7BA432C4974AF466A 1 038384CCFECD93EBC74AE497B5C59C5A1DFC75874214299690139347BCC09B29 2 095E45DDF417D05FB10933FFC63D474548B7FFFF7888802F07FFFFFF7D07A8A8"
            },
            "g_i": "1 0FC61615A31975C74D052AC7E246A556FC6D4696C58636B7BA432C4974AF466A 1 038384CCFECD93EBC74AE497B5C59C5A1DFC75874214299690139347BCC09B29 2 095E45DDF417D05FB10933FFC63D474548B7FFFF7888802F07FFFFFF7D07A8A8",
            "i": 1,
            "m2": "B668E0077478C6F6917F8B9A74F4DD73FC59007ACD8A0400D6A351A17501A340"
        }
    },
    "signature_correctness_proof": {...},
    "rev_reg": {
        "accum": "21 13DA29474FD59E4EC67972E1011683FBC7E29A4D84B471175E3DE0D6CA8614D8C 21 126294F88CFC2EDD4AB903E902CA89317524561C33AB3F8CCD1AF4EF37FFF61E5 6 640A166EAB12C87ECE0FFCFC1BB3F7F72571394592CF2642E771F2F64E2E2C6B 4 09DA376CB961CCC9746010A0EE4F160E2B5824B867BAE4439325ABD8BD40838C 6 79919234373422F54EFEA42DFB2A097352B9559E810DD50F0B8F6DC8F172F774 4 3CB1441C0F8DA480066096A2EB6CD4A3DD6AB004305E9990D31D39C3C30F326D"
    },
    "witness": {
        "omega": "21 13381DCAB0C6BD5988B1960D7B66F66582F47B44679D5D8673867ED3784DAF87A 21 1383A484A08E39DFD04CBE5C823D2B3E272F7250755C082E74DD118E91E12DB7F 6 8454CF94F3BA261605200CC69C47641BEAA67CEDE51DEDB2117758C6CACF6800 4 1F3B0E14082B3B1D6824FCE4CCAE671FD9E3686C59DE086C2AB9367BF861E508 6 5301D8A820EB55DCA4A3DD6D110C46C560B5A377EF53A36D7C01061B226210A7 4 46C6E745560E23A00062955C3D959EC7FDED4129793512595DFC71167A06B310"
    }
}
```

Additional attributes
* `signature`
  * `r_credential`: The revocation credential that includes values for revocation
* `rev_reg`: The non revocation registry ID of the revocation registry, the issued credentials is assigned. [TODO: add ref to revocation]
* `witness`: Witness information.  [TODO: add ref to revocation]