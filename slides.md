---
theme: eloc
title: Nix - Declare Everything
info: |
  对 Nix 的简单介绍
---

# `Nix`

*Declare everything*

---

```bash
$ brew install node
```

---

☹️: `node` is not reproducible...

```bash
# 2022
$ brew install node
==> Downloading https://ghcr.io/v2/homebrew/core/node/manifests/...

$ node -v
v18.0.1
```

```bash
# 2024
$ brew install node
==> Downloading https://ghcr.io/v2/homebrew/core/node/manifests/...

$ node -v
v20.15.1
```
---

```bash
$ brew install node
==> Downloading https://ghcr.io/v2/homebrew/core/node/manifests/22.7.0
#####################################################################100.0%
==> Fetching dependencies for node: brotli, c-ares, icu4c, openssl@3
==> Downloading https://ghcr.io/v2/homebrew/core/brotli/manifests/1.1.0-1
==> ...
```

---

```json
{
  "main": "src/index.js",
  "dependencies": {
    "@slidev/cli": "^0.49.24",
    "slidev-theme-eloc": "^1.0.2",
    "vue": "^3.4.38"
  }
}
```

---

$f(\text{source, dependency}) \rightarrow \text{package}$


---

```typescript
let func = (name) => `Hello ${name}`;

return func("World");
```
---

`Nix` the programming language

```nix
let
  f = name: "Hello ${name}!";
in
  f "World"
```

---

$f(\text{source, dependency}) \rightarrow \text{package}$

```nix
mkPackage = { source, dependency }: <Package>;
```


---

```nix
let
  mkPackage = { source, dependency }: <Package>;
in
  nodejs = mkPackage {
    source = ?
    dependency = ?
  }
```

---

```nix
let
  mkPackage = { source, dependency }: <Package>;
in
  nodejs = mkPackage {
    source = fetchFromGithub {
      owner = "nodejs";
      repo = "node";
      rev = "20.15.1";
      hash = "sha256-4q4LFsq4yU1xRwNsM1sJoNVphJCnxaVe2IyL6AeHJ/I=";
    };
    dependency = ?
  }
```

---

```nix
let
  mkPackage = { source, dependency }: <Package>;
in
  nodejs = mkPackage {
    source = fetchFromGithub {
      owner = "nodejs";
      repo = "node";
      rev = "20.15.1";
      hash = "sha256-4q4LFsq4yU1xRwNsM1sJoNVphJCnxaVe2IyL6AeHJ/I=";
    };
    dependency = [
      brotli
      openssl
      ...
    ];
  }
```

---

```nix
let
  mkPackage = { source, dependency }: <Package>;
in
  nodejs = mkPackage {
    source = fetchFromGithub {
      owner = "nodejs";
      repo = "node";
      rev = "20.15.1";
      hash = "sha256-4q4LFsq4yU1xRwNsM1sJoNVphJCnxaVe2IyL6AeHJ/I=";
    };
    dependency = [
      brotli = mkPackage { hash = "sha256-abc", ... };
      openssl = mkPackage { hash = "sha256-xyz", ... };
      ...
    ];
  }
```

---

**pinned** input -> **pinned** output

```
source      = sha256-4q4LFsq4yU1xRwNsJoNVphJCnxaVe2IyL6AeHJ/I=
dependency1 = sha256-zzCYlQy02FOtlcCEHx+cbT3BAtzPys1SHZOSUgi3asg=
dependency2 = sha256-GXFJwY2enyksQ/BACsq6EuX1LKz+BQ89GZJ36nOOwuc=
```
👇
```
result = vj9c8dm21k27xzbsaiy4j1zm6scb5nxk-nodejs-20.15.1
```

---

Every **build output** in Nix is a `Derivation`!

```
vj9c8dm21k27xzbsaiy4j1zm6scb5nxk-nodejs-20.15.1
```

---

😄

```bash
$ nix build .#nodejs

$ cd result && pwd
/nix/store/vj9c8dm21k27xzbsaiy4j1zm6scb5nxk-nodejs-20.15.1/

$ /nix/store/vj9c8dm21k27xzbsaiy4j1zm6scb5nxk-nodejs-20.15.1/bin/node -v
v20.15.1
```

---

🤔: `Node.js` + `pnpm` + `vim` + `docker` + `VSCode` + ...

---

☹️

```bash
$ brew install node

$ npm install -g pnpm

$ brew install docker

$ brew install --cask visual-studio-code
```

---

🤔
```
nodejs 👉 vj9c8dm21k27xzbsaiy4j1zm6scb5nxk-nodejs-20.15.1
pnpm   👉 jj8gfv9a1gvmss4c4mvbsr2v9sp8wn8l-pnpm-8.15.5
docker 👉 gmbhq7670yhjnj95cswn8v3907rvmmkr-docker-24.0.9
vscode 👉 y7kwc3ccy5mgwbvcf28mblrmrd1vn1im-vscode-1.91.1
```

---

$f(\text{Node.js + pnpm + vim + docker + ...}) \rightarrow \text{environment}$

---

$f(\text{packages}) \rightarrow \text{environment}$

---

```nix
mkEnvironment = { packages }: <Environment>;
```

---

```nix
let
  mkEnvironment = { packages }: <Environment>;
in
  mkEnvironment {
    packages = [
      nodejs pnpm vim docker
    ];
  }
```

---

😄

```bash
$ nix-shell environment.nix

$ node -v
v20.15.1

$ vim -v
v9.0.2142
```

---

Every **build output** in Nix is a `Derivation`!

```bash
$ nix-build environment.nix

$ cd result && pwd
/nix/store/gvrsj8az9aq7llv97h5mn0204v3n4qf9-nix-shell
```

---

🤔: `settings.json`? `.zshrc`? `.gitconfig`?

---

```nix
let
  mkEnvironment = { config }: <Environment>;
in
  mkEnvironment {
    config = {
      packages = [ ... ];
      gitconfig = {
        userName = "AsterisMono";
        userEmail = "cmiki@amono.me";
        extraConfig = {
          init.defaultBranch = "main";
        };
      };
    };
  }
```

---

😄

```bash
$ nix-shell environment.nix

$ git config user.name
AsterisMono
```

---

🤔: `postgresql`? `redis`? `rabbitmq`?

---

```nix
let
  mkEnvironment = { config }: <Environment>;
in
  mkEnvironment {
    config = {
      packages = [ ... ];
      gitconfig = { ... };
      services = {
        postgres = {
          enable = true;
          port = 5432;
        };
        redis.enable = true;
      };
    };
  }
```

---

😵‍💫

```
DATABASE_URL=?
REDIS_URL=?
REDIS_PORT=?
REDIS_PASSWORD=?
RABBITMQ_VHOST=?
RABBITMQ_USERNAME=?
```

---

```nix
let
  db_username = "postgres";
  db_password = "postgres";
  mkEnvironment = { config }: <Environment>;
in
  mkEnvironment {
    config = {
      services = {
        postgres = {
          enable = true;
          port = 5432;
          defaultUsername = db_username;
          defaultPassword = db_password;
        };
      };
      env = {
        DATABASE_URL = "postgres://${db_username}:${db_password}@localhost:5432/postgres?schema=public";
      };
    };
  }
```

---

😄

```bash
$ nix develop . --impure

# Service started 🔥

$ pnpm install && pnpm run test # pnpm/node ✅

$ code . # VSCode ✅

$ git commit -m "feat: xxx" # gitconfig ✅
```



---

Every **build output** in Nix is a `Derivation`
```nix
let
  # vj9c8dm21k27xzbsaiy4j1zm6scb5nxk-nodejs-20.15.1
  mkPackage = { source, dependency }@inputs: lib.mkDerivation { inputs };
  
  # 9szdqawq89qgpgsrh2nw44hxans5fqa6-devenv-profile
  mkEnvironment = { packages, services, config }@inputs: lib.mkDerivation { inputs };
in
  ...
```

Even... 👀
---

### NixOS <img style="float: right; margin-left: 16px" src="https://search.nixos.org/images/nix-logo.png" alt="drawing" width="88"/>

---