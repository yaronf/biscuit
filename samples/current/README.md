# Biscuit samples and expected results

root secret key: 12aca40167fbdd1a11037e9fd440e3d510d9d9dea70a6646aa4aaf84d718d75a
root public key: acdd6d5b53bfee478bf689f8e012fe7988bf755e3d7c5152947abc149bc20189

------------------------------

## basic token: test1_basic.bc
### token

authority:
symbols: ["file1", "file2"]

public keys: []

```
right("file1", "read");
right("file2", "read");
right("file1", "write");
```

1:
symbols: ["0"]

public keys: []

```
check if resource($0), operation("read"), right($0, "read");
```

### validation

authorizer code:
```
resource("file1");

allow if true;
```

revocation ids:
- `3ee1c0f42ba69ec63b1f39a6b3c57d25a4ccec452233ca6d40530ecfe83af4918fa78d9346f8b7c498545b54663960342b9ed298b2c8bbe2085b80c237b56f09`
- `e16ccf0820b02092adb531e36c2e82884c6c6c647b1c85184007f2ace601648afb71faa261b11f9ab352093c96187870f868588b664579c8018864b306bd5007`

authorizer world:
```
World {
  facts: {
    "resource(\"file1\")",
    "right(\"file1\", \"read\")",
    "right(\"file1\", \"write\")",
    "right(\"file2\", \"read\")",
}
  rules: {}
  checks: {}
  policies: {
    "allow if true",
}
}
```

result: `Err(FailedLogic(Unauthorized { policy: Allow(0), checks: [Block(FailedBlockCheck { block_id: 1, check_id: 0, rule: "check if resource($0), operation(\"read\"), right($0, \"read\")" })] }))`


------------------------------

## different root key: test2_different_root_key.bc
### token

authority:
symbols: ["file1"]

public keys: []

```
right("file1", "read");
```

1:
symbols: ["0"]

public keys: []

```
check if resource($0), operation("read"), right($0, "read");
```

### validation

result: `Err(Format(Signature(InvalidSignature("signature error: Verification equation was not satisfied"))))`


------------------------------

## invalid signature format: test3_invalid_signature_format.bc
### token

authority:
symbols: ["file1", "file2"]

public keys: []

```
right("file1", "read");
right("file2", "read");
right("file1", "write");
```

1:
symbols: ["0"]

public keys: []

```
check if resource($0), operation("read"), right($0, "read");
```

### validation

result: `Err(Format(InvalidSignatureSize(16)))`


------------------------------

## random block: test4_random_block.bc
### token

authority:
symbols: ["file1", "file2"]

public keys: []

```
right("file1", "read");
right("file2", "read");
right("file1", "write");
```

1:
symbols: ["0"]

public keys: []

```
check if resource($0), operation("read"), right($0, "read");
```

### validation

result: `Err(Format(Signature(InvalidSignature("signature error: Verification equation was not satisfied"))))`


------------------------------

## invalid signature: test5_invalid_signature.bc
### token

authority:
symbols: ["file1", "file2"]

public keys: []

```
right("file1", "read");
right("file2", "read");
right("file1", "write");
```

1:
symbols: ["0"]

public keys: []

```
check if resource($0), operation("read"), right($0, "read");
```

### validation

result: `Err(Format(Signature(InvalidSignature("signature error: Verification equation was not satisfied"))))`


------------------------------

## reordered blocks: test6_reordered_blocks.bc
### token

authority:
symbols: ["file1", "file2"]

public keys: []

```
right("file1", "read");
right("file2", "read");
right("file1", "write");
```

1:
symbols: ["0"]

public keys: []

```
check if resource($0), operation("read"), right($0, "read");
```

2:
symbols: []

public keys: []

```
check if resource("file1");
```

### validation

result: `Err(Format(Signature(InvalidSignature("signature error: Verification equation was not satisfied"))))`


------------------------------

## scoped rules: test7_scoped_rules.bc
### token

authority:
symbols: ["user_id", "alice", "file1"]

public keys: []

```
user_id("alice");
owner("alice", "file1");
```

1:
symbols: ["0", "1"]

public keys: []

```
right($0, "read") <- resource($0), user_id($1), owner($1, $0);
check if resource($0), operation("read"), right($0, "read");
```

2:
symbols: ["file2"]

public keys: []

```
owner("alice", "file2");
```

### validation

authorizer code:
```
resource("file2");
operation("read");

allow if true;
```

revocation ids:
- `02d287b0e5b22780192f8351538583c17f7d0200e064b32a1fcf07899e64ffb10e4de324f5c5ebc72c89a63e424317226cf555eb42dae81b2fd4639cf7591108`
- `22e75ea200cf7b2b62b389298fe0dec973b7f9c7e54e76c3c41811d72ea82c68227bc9079b7d05986de17ef9301cccdc08f5023455386987d1e6ee4391b19f06`
- `140a3631fecae550b51e50b9b822b947fb485c80070b34482fa116cdea560140164a1d0a959b40fed8a727e2f62c0b57635760c488c8bf0eda80ee591558c409`

authorizer world:
```
World {
  facts: {
    "operation(\"read\")",
    "owner(\"alice\", \"file1\")",
    "owner(\"alice\", \"file2\")",
    "resource(\"file2\")",
    "user_id(\"alice\")",
}
  rules: {
    "right($0, \"read\") <- resource($0), user_id($1), owner($1, $0)",
}
  checks: {}
  policies: {
    "allow if true",
}
}
```

result: `Err(FailedLogic(Unauthorized { policy: Allow(0), checks: [Block(FailedBlockCheck { block_id: 1, check_id: 0, rule: "check if resource($0), operation(\"read\"), right($0, \"read\")" })] }))`


------------------------------

## scoped checks: test8_scoped_checks.bc
### token

authority:
symbols: ["file1"]

public keys: []

```
right("file1", "read");
```

1:
symbols: ["0"]

public keys: []

```
check if resource($0), operation("read"), right($0, "read");
```

2:
symbols: ["file2"]

public keys: []

```
right("file2", "read");
```

### validation

authorizer code:
```
resource("file2");
operation("read");

allow if true;
```

revocation ids:
- `567682495bf002eb84c46491e40fad8c55943d918c65e2c110b1b88511bf393072c0305a243e3d632ca5f1e9b0ace3e3582de84838c3a258480657087c267f02`
- `71f0010b1034dbc62c53f67a23947b92ccba46495088567ac7ad5c4d7d65476964bee42053a6a35088110c5918f9c9606057689271fef89d84253cf98e6d4407`
- `6d00d5f2a5d25dbfaa19152a81b44328b368e8fb8300b25e36754cfe8b2ce1eb2d1452ce9b1502e6f377a23aa87098fb05b5b073541624a8815ba0610f793005`

authorizer world:
```
World {
  facts: {
    "operation(\"read\")",
    "resource(\"file2\")",
    "right(\"file1\", \"read\")",
    "right(\"file2\", \"read\")",
}
  rules: {}
  checks: {}
  policies: {
    "allow if true",
}
}
```

result: `Err(FailedLogic(Unauthorized { policy: Allow(0), checks: [Block(FailedBlockCheck { block_id: 1, check_id: 0, rule: "check if resource($0), operation(\"read\"), right($0, \"read\")" })] }))`


------------------------------

## expired token: test9_expired_token.bc
### token

authority:
symbols: []

public keys: []

```
```

1:
symbols: ["file1", "expiration"]

public keys: []

```
check if resource("file1");
check if time($time), $time <= 2018-12-20T00:00:00Z;
```

### validation

authorizer code:
```
resource("file1");
operation("read");
time(2020-12-21T09:23:12Z);

allow if true;
```

revocation ids:
- `b2474f3e0a5788cdeff811f2599497a04d1ad71ca48dbafb90f20a950d565dda0b86bd6c9072a727c19b6b20a1ae10d8cb88155186550b77016ffd1dca9a6203`
- `0d12152670cbefe2fa504af9a92b513f1a48ae460ae5e66aaac4ed9f7dc3cc1c4c510693312b351465062169a2169fc520ce4e17e548d21982c81a74c66a3c0c`

authorizer world:
```
World {
  facts: {
    "operation(\"read\")",
    "resource(\"file1\")",
    "time(2020-12-21T09:23:12Z)",
}
  rules: {}
  checks: {}
  policies: {
    "allow if true",
}
}
```

result: `Err(FailedLogic(Unauthorized { policy: Allow(0), checks: [Block(FailedBlockCheck { block_id: 1, check_id: 1, rule: "check if time($time), $time <= 2018-12-20T00:00:00Z" })] }))`


------------------------------

## authorizer scope: test10_authorizer_scope.bc
### token

authority:
symbols: ["file1"]

public keys: []

```
right("file1", "read");
```

1:
symbols: ["file2"]

public keys: []

```
right("file2", "read");
```

### validation

authorizer code:
```
resource("file2");
operation("read");

check if right($0, $1), resource($0), operation($1);

allow if true;
```

revocation ids:
- `b9ecf192ecb1bbb10e45320c1c86661f0c6b6bd28e89fdd8fa838fe0ab3f754229f7fbbf92ad978d36f744c345c69bc156a2a91a2979a3c235a9d936d401b404`
- `839728735701e589c2612e655afa2b53f573480e6a0477ae68ed71587987d1af398a31296bdec0b6eccee9348f4b4c23ca1031e809991626c579fef80b1d380d`

authorizer world:
```
World {
  facts: {
    "operation(\"read\")",
    "resource(\"file2\")",
    "right(\"file1\", \"read\")",
    "right(\"file2\", \"read\")",
}
  rules: {}
  checks: {
    "check if right($0, $1), resource($0), operation($1)",
}
  policies: {
    "allow if true",
}
}
```

result: `Err(FailedLogic(Unauthorized { policy: Allow(0), checks: [Authorizer(FailedAuthorizerCheck { check_id: 0, rule: "check if right($0, $1), resource($0), operation($1)" })] }))`


------------------------------

## authorizer authority checks: test11_authorizer_authority_caveats.bc
### token

authority:
symbols: ["file1"]

public keys: []

```
right("file1", "read");
```

### validation

authorizer code:
```
resource("file2");
operation("read");

check if right($0, $1), resource($0), operation($1);

allow if true;
```

revocation ids:
- `593d273d141bf23a3e89b55fffe1b3f96f683a022bb763e78f4e49f31a7cf47668c3fd5e0f580727ac9113ede302d34264597f6f1e6c6dd4167836d57aedf504`

authorizer world:
```
World {
  facts: {
    "operation(\"read\")",
    "resource(\"file2\")",
    "right(\"file1\", \"read\")",
}
  rules: {}
  checks: {
    "check if right($0, $1), resource($0), operation($1)",
}
  policies: {
    "allow if true",
}
}
```

result: `Err(FailedLogic(Unauthorized { policy: Allow(0), checks: [Authorizer(FailedAuthorizerCheck { check_id: 0, rule: "check if right($0, $1), resource($0), operation($1)" })] }))`


------------------------------

## authority checks: test12_authority_caveats.bc
### token

authority:
symbols: ["file1"]

public keys: []

```
check if resource("file1");
```

### validation for "file1"

authorizer code:
```
resource("file1");
operation("read");

allow if true;
```

revocation ids:
- `0a1d14a145debbb0a2f4ce0631d3a0a48a2e0eddabefda7fabb0414879ec6be24b9ae7295c434609ada3f8cc47b8845bbd5a0d4fba3d96748ff1b824496e0405`

authorizer world:
```
World {
  facts: {
    "operation(\"read\")",
    "resource(\"file1\")",
}
  rules: {}
  checks: {}
  policies: {
    "allow if true",
}
}
```

result: `Ok(0)`
### validation for "file2"

authorizer code:
```
resource("file2");
operation("read");

allow if true;
```

revocation ids:
- `0a1d14a145debbb0a2f4ce0631d3a0a48a2e0eddabefda7fabb0414879ec6be24b9ae7295c434609ada3f8cc47b8845bbd5a0d4fba3d96748ff1b824496e0405`

authorizer world:
```
World {
  facts: {
    "operation(\"read\")",
    "resource(\"file2\")",
}
  rules: {}
  checks: {}
  policies: {
    "allow if true",
}
}
```

result: `Err(FailedLogic(Unauthorized { policy: Allow(0), checks: [Block(FailedBlockCheck { block_id: 0, check_id: 0, rule: "check if resource(\"file1\")" })] }))`


------------------------------

## block rules: test13_block_rules.bc
### token

authority:
symbols: ["file1", "file2"]

public keys: []

```
right("file1", "read");
right("file2", "read");
```

1:
symbols: ["valid_date", "0", "1"]

public keys: []

```
valid_date("file1") <- time($0), resource("file1"), $0 <= 2030-12-31T12:59:59Z;
valid_date($1) <- time($0), resource($1), $0 <= 1999-12-31T12:59:59Z, !["file1"].contains($1);
check if valid_date($0), resource($0);
```

### validation for "file1"

authorizer code:
```
resource("file1");
time(2020-12-21T09:23:12Z);

allow if true;
```

revocation ids:
- `d251352efd4e4c72e8a1609fce002f558f1a0bb5e36cd3d8b3a6c6599e3960880f21bea6fe1857f4ecbc2c399dd77829b154e75f1323e9dec413aad70f97650d`
- `9de4f51e6019540598a957515dad52f5403e5c6cd8d2adbca1bff42a4fbc0eb8c6adab499da2fe894a8a9c9c581276bfb0fdc3d35ab2ff9f920a2c4690739903`

authorizer world:
```
World {
  facts: {
    "resource(\"file1\")",
    "right(\"file1\", \"read\")",
    "right(\"file2\", \"read\")",
    "time(2020-12-21T09:23:12Z)",
    "valid_date(\"file1\")",
}
  rules: {
    "valid_date(\"file1\") <- time($0), resource(\"file1\"), $0 <= 2030-12-31T12:59:59Z",
    "valid_date($1) <- time($0), resource($1), $0 <= 1999-12-31T12:59:59Z, ![\"file1\"].contains($1)",
}
  checks: {}
  policies: {
    "allow if true",
}
}
```

result: `Ok(0)`
### validation for "file2"

authorizer code:
```
resource("file2");
time(2020-12-21T09:23:12Z);

allow if true;
```

revocation ids:
- `d251352efd4e4c72e8a1609fce002f558f1a0bb5e36cd3d8b3a6c6599e3960880f21bea6fe1857f4ecbc2c399dd77829b154e75f1323e9dec413aad70f97650d`
- `9de4f51e6019540598a957515dad52f5403e5c6cd8d2adbca1bff42a4fbc0eb8c6adab499da2fe894a8a9c9c581276bfb0fdc3d35ab2ff9f920a2c4690739903`

authorizer world:
```
World {
  facts: {
    "resource(\"file2\")",
    "right(\"file1\", \"read\")",
    "right(\"file2\", \"read\")",
    "time(2020-12-21T09:23:12Z)",
}
  rules: {
    "valid_date(\"file1\") <- time($0), resource(\"file1\"), $0 <= 2030-12-31T12:59:59Z",
    "valid_date($1) <- time($0), resource($1), $0 <= 1999-12-31T12:59:59Z, ![\"file1\"].contains($1)",
}
  checks: {}
  policies: {
    "allow if true",
}
}
```

result: `Err(FailedLogic(Unauthorized { policy: Allow(0), checks: [Block(FailedBlockCheck { block_id: 1, check_id: 0, rule: "check if valid_date($0), resource($0)" })] }))`


------------------------------

## regex_constraint: test14_regex_constraint.bc
### token

authority:
symbols: ["0", "file[0-9]+.txt"]

public keys: []

```
check if resource($0), $0.matches("file[0-9]+.txt");
```

### validation for "file1"

authorizer code:
```
resource("file1");

allow if true;
```

revocation ids:
- `1c158e1e12c8670d3f4411597276fe1caab17b7728adb7f7e9c44eeec3e3d85676e6ebe2d28c287e285a45912386cfa53e1752997630bd7a4ca6c2cd9f143500`

authorizer world:
```
World {
  facts: {
    "resource(\"file1\")",
}
  rules: {}
  checks: {}
  policies: {
    "allow if true",
}
}
```

result: `Err(FailedLogic(Unauthorized { policy: Allow(0), checks: [Block(FailedBlockCheck { block_id: 0, check_id: 0, rule: "check if resource($0), $0.matches(\"file[0-9]+.txt\")" })] }))`
### validation for "file123"

authorizer code:
```
resource("file123.txt");

allow if true;
```

revocation ids:
- `1c158e1e12c8670d3f4411597276fe1caab17b7728adb7f7e9c44eeec3e3d85676e6ebe2d28c287e285a45912386cfa53e1752997630bd7a4ca6c2cd9f143500`

authorizer world:
```
World {
  facts: {
    "resource(\"file123.txt\")",
}
  rules: {}
  checks: {}
  policies: {
    "allow if true",
}
}
```

result: `Ok(0)`


------------------------------

## multi queries checks: test15_multi_queries_caveats.bc
### token

authority:
symbols: ["must_be_present", "hello"]

public keys: []

```
must_be_present("hello");
```

### validation

authorizer code:
```
check if must_be_present($0) or must_be_present($0);

allow if true;
```

revocation ids:
- `d3eee8a74eacec9c51d4d1eb29b479727dfaafa9df7d4c651d07c493c56f3a5f037a51139ebd036f50d1159d12bccec3e377bbd32db90a39dd52c4776757ad0b`

authorizer world:
```
World {
  facts: {
    "must_be_present(\"hello\")",
}
  rules: {}
  checks: {
    "check if must_be_present($0) or must_be_present($0)",
}
  policies: {
    "allow if true",
}
}
```

result: `Ok(0)`


------------------------------

## check head name should be independent from fact names: test16_caveat_head_name.bc
### token

authority:
symbols: ["hello"]

public keys: []

```
check if resource("hello");
```

1:
symbols: ["test"]

public keys: []

```
query("test");
```

### validation

authorizer code:
```
allow if true;
```

revocation ids:
- `e79679e019f1d7d3a9f9a309673aceadc7b2b2d67c0df3e7a1dccec25218e9b5935b9c8f8249243446406e3cdd86c1b35601a21cf1b119df48ca5e897cc6cd0d`
- `2042ea2dca41ba3eb31196f49b211e615dcba46067be126e6035b8549bb57cdfeb24d07f2b44241bc0f70cc8ddc31e30772116d785b82bc91be8440dfdab500f`

authorizer world:
```
World {
  facts: {
    "query(\"test\")",
}
  rules: {}
  checks: {}
  policies: {
    "allow if true",
}
}
```

result: `Err(FailedLogic(Unauthorized { policy: Allow(0), checks: [Block(FailedBlockCheck { block_id: 0, check_id: 0, rule: "check if resource(\"hello\")" })] }))`


------------------------------

## test expression syntax and all available operations: test17_expressions.bc
### token

authority:
symbols: ["hello world", "hello", "world", "aaabde", "a*c?.e", "abd", "aaa", "b", "de", "abcD12", "abc", "def"]

public keys: []

```
check if true;
check if !false;
check if !false && true;
check if false or true;
check if (true || false) && true;
check if 1 < 2;
check if 2 > 1;
check if 1 <= 2;
check if 1 <= 1;
check if 2 >= 1;
check if 2 >= 2;
check if 3 == 3;
check if 1 + 2 * 3 - 4 / 2 == 5;
check if "hello world".starts_with("hello") && "hello world".ends_with("world");
check if "aaabde".matches("a*c?.e");
check if "aaabde".contains("abd");
check if "aaabde" == "aaa" + "b" + "de";
check if "abcD12" == "abcD12";
check if 2019-12-04T09:46:41Z < 2020-12-04T09:46:41Z;
check if 2020-12-04T09:46:41Z > 2019-12-04T09:46:41Z;
check if 2019-12-04T09:46:41Z <= 2020-12-04T09:46:41Z;
check if 2020-12-04T09:46:41Z >= 2020-12-04T09:46:41Z;
check if 2020-12-04T09:46:41Z >= 2019-12-04T09:46:41Z;
check if 2020-12-04T09:46:41Z >= 2020-12-04T09:46:41Z;
check if 2020-12-04T09:46:41Z == 2020-12-04T09:46:41Z;
check if hex:12ab == hex:12ab;
check if [1, 2].contains(2);
check if [2019-12-04T09:46:41Z, 2020-12-04T09:46:41Z].contains(2020-12-04T09:46:41Z);
check if [false, true].contains(true);
check if ["abc", "def"].contains("abc");
check if [hex:12ab, hex:34de].contains(hex:34de);
```

### validation

authorizer code:
```
allow if true;
```

revocation ids:
- `f1391d7549f4c165dd58878c8433bdc23634533f4910bcd37fc53d5b3b29c542ef1b46a38a42beb2df911d526a52f65774e1d60f910dbe15ca1591735617240b`

authorizer world:
```
World {
  facts: {}
  rules: {}
  checks: {}
  policies: {
    "allow if true",
}
}
```

result: `Ok(0)`


------------------------------

## invalid block rule with unbound_variables: test18_unbound_variables_in_rule.bc
### token

authority:
symbols: []

public keys: []

```
check if operation("read");
```

1:
symbols: ["unbound", "any1", "any2"]

public keys: []

```
operation($unbound, "read") <- operation($any1, $any2);
```

### validation

result: `Err(FailedLogic(InvalidBlockRule(0, "operation($unbound, \"read\") <- operation($any1, $any2)")))`


------------------------------

## invalid block rule generating an #authority or #ambient symbol with a variable: test19_generating_ambient_from_variables.bc
### token

authority:
symbols: []

public keys: []

```
check if operation("read");
```

1:
symbols: ["any"]

public keys: []

```
operation("read") <- operation($any);
```

### validation

authorizer code:
```
operation("write");

allow if true;
```

revocation ids:
- `4819e7360fdb840e54e94afcbc110e9b0652894dba2b8bf3b8b8f2254aaf00272bba7eb603c153c7e50cca0e5bb8e20449d70a1b24e7192e902c64f94848a703`
- `4a4c59354354d2f91b3a2d1e7afa2c5eeaf8be9f7b163c6b9091817551cc8661f0f3e0523b525ef9a5e597c0dd1f32e09e97ace531c150dba335bb3e1d329d00`

authorizer world:
```
World {
  facts: {
    "operation(\"read\")",
    "operation(\"write\")",
}
  rules: {
    "operation(\"read\") <- operation($any)",
}
  checks: {}
  policies: {
    "allow if true",
}
}
```

result: `Err(FailedLogic(Unauthorized { policy: Allow(0), checks: [Block(FailedBlockCheck { block_id: 0, check_id: 0, rule: "check if operation(\"read\")" })] }))`


------------------------------

## sealed token: test20_sealed.bc
### token

authority:
symbols: ["file1", "file2"]

public keys: []

```
right("file1", "read");
right("file2", "read");
right("file1", "write");
```

1:
symbols: ["0"]

public keys: []

```
check if resource($0), operation("read"), right($0, "read");
```

### validation

authorizer code:
```
resource("file1");
operation("read");

allow if true;
```

revocation ids:
- `b279f8c6fee5ea3c3fcb5109d8c6b35ba3fecea64d83a4dc387102b9401633a1558ac6ac50ddd7fd9e9877f936f9f4064abd467faeca2bef3114b9695eb0580e`
- `e1f0aca12704c1a3b9bb6292504ca6070462d9e043756dd209e625084e7d4053078bd4e55b6eebebbeb771d26d7794aa95f6b39ff949431548b32585a7379f0c`

authorizer world:
```
World {
  facts: {
    "operation(\"read\")",
    "resource(\"file1\")",
    "right(\"file1\", \"read\")",
    "right(\"file1\", \"write\")",
    "right(\"file2\", \"read\")",
}
  rules: {}
  checks: {}
  policies: {
    "allow if true",
}
}
```

result: `Ok(0)`


------------------------------

## parsing: test21_parsing.bc
### token

authority:
symbols: ["ns::fact_123", "hello é\t😁"]

public keys: []

```
ns::fact_123("hello é	😁");
```

### validation

authorizer code:
```
check if ns::fact_123("hello é	😁");

allow if true;
```

revocation ids:
- `4797a528328c8b5fb7939cc8956d8cda2513f552466eee501e26ea13a6cf6b4a381fd74ae547a9b50b627825142287d899b9d7bd1b5cfb18664a1be78320ea06`

authorizer world:
```
World {
  facts: {
    "ns::fact_123(\"hello é\t😁\")",
}
  rules: {}
  checks: {
    "check if ns::fact_123(\"hello é\t😁\")",
}
  policies: {
    "allow if true",
}
}
```

result: `Ok(0)`


------------------------------

## default_symbols: test22_default_symbols.bc
### token

authority:
symbols: []

public keys: []

```
read(0);
write(1);
resource(2);
operation(3);
right(4);
time(5);
role(6);
owner(7);
tenant(8);
namespace(9);
user(10);
team(11);
service(12);
admin(13);
email(14);
group(15);
member(16);
ip_address(17);
client(18);
client_ip(19);
domain(20);
path(21);
version(22);
cluster(23);
node(24);
hostname(25);
nonce(26);
query(27);
```

### validation

authorizer code:
```
check if read(0), write(1), resource(2), operation(3), right(4), time(5), role(6), owner(7), tenant(8), namespace(9), user(10), team(11), service(12), admin(13), email(14), group(15), member(16), ip_address(17), client(18), client_ip(19), domain(20), path(21), version(22), cluster(23), node(24), hostname(25), nonce(26), query(27);

allow if true;
```

revocation ids:
- `38094260b324eff92db2ef79e715d88c18503c0dafa400bff900399f2ab0840cedc5ac25bdd3e97860b3f9e78ca5e0df67a113eb87be50265d49278efb13210f`

authorizer world:
```
World {
  facts: {
    "admin(13)",
    "client(18)",
    "client_ip(19)",
    "cluster(23)",
    "domain(20)",
    "email(14)",
    "group(15)",
    "hostname(25)",
    "ip_address(17)",
    "member(16)",
    "namespace(9)",
    "node(24)",
    "nonce(26)",
    "operation(3)",
    "owner(7)",
    "path(21)",
    "query(27)",
    "read(0)",
    "resource(2)",
    "right(4)",
    "role(6)",
    "service(12)",
    "team(11)",
    "tenant(8)",
    "time(5)",
    "user(10)",
    "version(22)",
    "write(1)",
}
  rules: {}
  checks: {
    "check if read(0), write(1), resource(2), operation(3), right(4), time(5), role(6), owner(7), tenant(8), namespace(9), user(10), team(11), service(12), admin(13), email(14), group(15), member(16), ip_address(17), client(18), client_ip(19), domain(20), path(21), version(22), cluster(23), node(24), hostname(25), nonce(26), query(27)",
}
  policies: {
    "allow if true",
}
}
```

result: `Ok(0)`


------------------------------

## execution scope: test23_execution_scope.bc
### token

authority:
symbols: ["authority_fact"]

public keys: []

```
authority_fact(1);
```

1:
symbols: ["block1_fact"]

public keys: []

```
block1_fact(1);
```

2:
symbols: ["var"]

public keys: []

```
check if authority_fact($var);
check if block1_fact($var);
```

### validation

authorizer code:
```
allow if true;
```

revocation ids:
- `6a3606836bc63b858f96ce5000c9bead8eda139ab54679a2a8d7a9984c2e5d864b93280acc1b728bed0be42b5b1c3be10f48a13a4dbd05fd5763de5be3855108`
- `5f1468fc60999f22c4f87fa088a83961188b4e654686c5b04bdc977b9ff4666d51a3d8be5594f4cef08054d100f31d1637b50bb394de7cccafc643c9b650390b`
- `3eda05ddb65ee90d715cefc046837c01de944d8c4a7ff67e3d9a9d8470b5e214a20a8b9866bfe5e0d385e530b75ec8fcfde46b7dd6d4d6647d1e955c9d2fb90d`

authorizer world:
```
World {
  facts: {
    "authority_fact(1)",
    "block1_fact(1)",
}
  rules: {}
  checks: {}
  policies: {
    "allow if true",
}
}
```

result: `Err(FailedLogic(Unauthorized { policy: Allow(0), checks: [Block(FailedBlockCheck { block_id: 2, check_id: 1, rule: "check if block1_fact($var)" })] }))`


------------------------------

## third party: test24_third_party.bc
### token

authority:
symbols: []

public keys: ["ed25519/a424157b8c00c25214ea39894bf395650d88426147679a9dd43a64d65ae5bc25"]

```
right("read");
check if group("admin") trusting ed25519/a424157b8c00c25214ea39894bf395650d88426147679a9dd43a64d65ae5bc25;
```

1:
symbols: []

public keys: []

external signature by: "ed25519/a424157b8c00c25214ea39894bf395650d88426147679a9dd43a64d65ae5bc25"

```
group("admin");
check if right("read");
```

### validation

authorizer code:
```
allow if true;
```

revocation ids:
- `4f61f2f2f9cefdcad03a82803638e459bef70d6fd72dbdf2bdcab78fbd23f33146e4ff9700e23acb547b820b871fa9b9fd3bb6d7a1a755afce47e9907c65600c`
- `683b23943b73f53f57f473571ba266f79f1fca0633be249bc135054371a11ffb101c57150ab2f1b9a6a160b45d09567a314b7dbc84224edf6188afd5b86d9305`

authorizer world:
```
World {
  facts: {
    "group(\"admin\")",
    "right(\"read\")",
}
  rules: {}
  checks: {}
  policies: {
    "allow if true",
}
}
```

result: `Ok(0)`


------------------------------

## block rules: test25_check_all.bc
### token

authority:
symbols: ["allowed_operations", "A", "B", "op", "allowed"]

public keys: []

```
allowed_operations(["A", "B"]);
check all operation($op), allowed_operations($allowed), $allowed.contains($op);
```

### validation for "A, B"

authorizer code:
```
operation("A");
operation("B");

allow if true;
```

revocation ids:
- `b4ee591001e4068a7ee8efb7a0586c3ca3a785558f34d1fa8dbfa21b41ace70de0b670ac49222c7413066d0d83e6d9edee94fb0fda4b27ea11e837304dfb4b0b`

authorizer world:
```
World {
  facts: {
    "allowed_operations([ \"A\", \"B\"])",
    "operation(\"A\")",
    "operation(\"B\")",
}
  rules: {}
  checks: {}
  policies: {
    "allow if true",
}
}
```

result: `Ok(0)`
### validation for "A, inalid"

authorizer code:
```
operation("A");
operation("invalid");

allow if true;
```

revocation ids:
- `b4ee591001e4068a7ee8efb7a0586c3ca3a785558f34d1fa8dbfa21b41ace70de0b670ac49222c7413066d0d83e6d9edee94fb0fda4b27ea11e837304dfb4b0b`

authorizer world:
```
World {
  facts: {
    "allowed_operations([ \"A\", \"B\"])",
    "operation(\"A\")",
    "operation(\"invalid\")",
}
  rules: {}
  checks: {}
  policies: {
    "allow if true",
}
}
```

result: `Err(FailedLogic(Unauthorized { policy: Allow(0), checks: [Block(FailedBlockCheck { block_id: 0, check_id: 0, rule: "check all operation($op), allowed_operations($allowed), $allowed.contains($op)" })] }))`
