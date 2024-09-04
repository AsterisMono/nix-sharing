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

😄

```bash
$ nix build .#nodejs

$ cd result && pwd
/nix/store/vj9c8dm21k27xzbsaiy4j1zm6scb5nxk-nodejs-20.15.1/

$ /nix/store/vj9c8dm21k27xzbsaiy4j1zm6scb5nxk-nodejs-20.15.1/bin/node -v
v20.15.1
```

---

Every **build output** in Nix is a `Derivation`!

```
vj9c8dm21k27xzbsaiy4j1zm6scb5nxk-nodejs-20.15.1
```

---

🪄: Run software **without installing**

```
$ nix-shell -p cowsay
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

```
mkPackage = { source, dependency }: <Package>;
```
👇
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
disabled: true
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

<div style="display: flex; align-items: center; gap: 16px; margin-bottom: 32px; margin-left: 30px;">
  <img style="margin-bottom: 10px; margin-right: 16px;" src="https://devenv.sh/assets/logo.webp" alt="drawing" width="350"/> <span>×</span> <img src="./logo.png" width="420" >
</div>

```bash
$ git clone https://github.com/frontonly/xycatalog-server

$ direnv allow

$ devenv up

$ pnpm run test # 🥳
```

---
disabled: true
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

📦💻🤔

---

### nix-darwin <img style="float: right; margin-left: 16px" src="https://camo.githubusercontent.com/72a8cc4dd5e3137b74db5fd82e693d3481934ee408620d5a75d9c302c4907f15/68747470733a2f2f646169646572642e636f6d2f6e69782d64617277696e2f696d616765732f6e69782d64617277696e2e706e67" alt="drawing" width="88"/>

*Declarative configuration for your MacOS devices*

---

```nix
defaults = {
  menuExtraClock.Show24Hour = true; # 24小时时钟

  dock = {
    autohide = true;      # 自动隐藏Dock
    show-recents = false; # 不显示最近运行的程序
    tilesize = 56;        # 图标大小
    mineffect = "scale";  # 缩放效果
  };

  universalaccess.reduceMotion = true; # 减少动画效果

  NSGlobalDomain = {
    ApplePressAndHoldEnabled = false; # 启用重复键
    KeyRepeat = 2;                    # 设置键重复速度
    InitialKeyRepeat = 25;
  };
};

```

---

```nix
homebrew = {
  enable = true;

  masApps = {
    TencentMeeting = 1484048379;
    Wechat = 836500024;
    NeteaseCloudMusic = 944848654;
    QQ = 451108668;
  };

  casks = [
    "visual-studio-code"
    "cursor"
    "google-chrome"
    ...
  ];
}
```

---

😎🍎

```bash
$ darwin-rebuild switch --flake .

building the system configuration...
restarting Dock...
setting up user launchd services...
Homebrew bundle...
Starting Home Manager activation
...
```

---

### home-manager <img style="float: right; margin-left: 16px" src="https://avatars.githubusercontent.com/u/33221035?s=200&v=4" alt="drawing" width="88"/>

*Declarative configuration for your `/home`*

---

```nix
programs.fish = {
  enable = true;
  plugins = [
    {
      name = "plugin-git";
      inherit (pkgs.fishPlugins.plugin-git) src;
    }
  ];
  shellAliases = {
    ".." = "cd ../";
    "n" = "nvim";
    "ls" = "eza -l";
  };
};
```

---

```nix
# Neovim
programs.neovim = {
  enable = true;
  defaultEditor = true;
  vimAlias = true;
};

xdg.configFile.nvim.source = flake.inputs.nvim-config;
```

---

```nix
programs.git = {
  enable = true;
  userName = "Chatnoir Miki";
  userEmail = "cmiki@amono.me";
  extraConfig = {
    init.defaultBranch = "main";
    pull.rebase = true;
  };
};
```

---

```nix
programs.firefox = {
  policies = {
    ExtensionSettings = {                  # 自动安装这些扩展
      "uBlock0@raymondhill.net" = {
        install_url = "https://addons.mozilla.org/firefox/downloads/latest/ublock-origin/latest.xpi";
        installation_mode = "force_installed";
        default_area = "menupanel";
      };
    };
  };
  DisplayBookmarksToolbar = "never";       # 不显示书签栏
  DisablePocket = true;                    # 关闭 Pocket
  Homepage.StartPage = "previous-session"; # 继续上一次浏览的位置
  profiles.default = {
    search.default = "Google";             # 默认谷歌搜索
  };
};
```

---

```nix
programs.vscode = {
  enable = true;
  extensions = with pkgs.vscode-extensions; [
    dracula-theme.theme-dracula
    vscodevim.vim
    yzhang.markdown-all-in-one
  ];
};
```

---

😏

---

### <img src="https://upload.wikimedia.org/wikipedia/commons/thumb/c/c4/NixOS_logo.svg/1600px-NixOS_logo.svg.png?20200523080210" alt="drawing" width="400"/>

 *Declarative configuration for your **systems***

---

💻 Desktop config

```nix
{
  services.clashMeta = {
    enable = true;
    tunMode = true;
    configFile = "${secrets}/clashConfig.yaml";
  };

  fonts = {
    fontDir.enable = true;
    packages = with pkgs; [
      (nerdfonts.override {
        fonts = [ "FiraCode" "Hack" "JetBrainsMono" "UbuntuMono" ];
      })
    ];
  };

  virtualisation.docker.enable = true;
}
```

---

🖥 Server config

```nix
{
  services.openssh = {
    enable = true;
    settings.PasswordAuthentication = false;
    settings.KbdInteractiveAuthentication = false;
    settings.PermitRootLogin = "without-password";
  };

  networking.firewall.enable = true;

  config.services.postgresql = {
    enable = true;
    ensureDatabases = [ "postgres" ];
  };
}
```

---

### <img src="https://github.com/serokell/deploy-rs/raw/master/docs/logo.svg" alt="drawing" width="300"/>

```nix
deploy.nodes = {
  celestia = {
    hostname = "celestia";
    profiles.system = {
      user = "root";
      path = activate.nixos self.nixosConfigurations.celestia;
    };
  };
  aqua = { ... };
};
```

---

#### Declarative.<br />Deterministic.<br/>Reproducible.

---

#### Get ❄️ [here](https://determinate.systems/posts/graphical-nix-installer/).

---

More ❄️ at [@nix_resources](https://linktr.ee/nix_resources)

Also checkout my [flake](https://github.com/AsterisMono/flake)!