# docker buildの様子

ともかくstraceはとってみました

https://gist.github.com/SpringMT/64439a2e51bb89d92c2fd8e0f7dc4198 ← docker クライアント

https://gist.github.com/SpringMT/148d686c7a9d13917199af52350b4f7e ← dockerd

straceみてもあんまりよくわからない。

## docker デーモンのstorage driver
https://docs.docker.com/storage/storagedriver/select-storage-driver/

storage-driverとしてデフォルトはoverlay2が指定されます。

https://docs.docker.com/storage/storagedriver/overlayfs-driver/

そのため、 /var/lib/docker/overlay2/ ディレクトリ配下にレイヤーのファイル郡ができます。

https://github.com/moby/moby/blob/6c0a036dce2051cc0d73221965e76e3d2e6a8a3a/builder/dockerfile/builder.go#L134

実際の処理としてはここらへんが走るはず。

https://github.com/moby/moby/blob/6c0a036dce2051cc0d73221965e76e3d2e6a8a3a/builder/dockerfile/dispatchers.go#L94

実際には、ハッシュ値をみながら、処理自体を走らせるかを確認した上で、FROMであればDLしたファイルなどを展開して保存します。

/var/lib/docker/overlay2/ 配下に ハッシュ値を元にしたdirが作られ、そのdirの中のdiffディレクトリに、自身のコンテンツが含まれます。

例えば下記のようなDockerfileをビルドしようとします。

```
FROM nginx:alpine

COPY default.conf /etc/nginx/conf.d/
COPY index.html /usr/share/nginx/html/
```

`COPY index.html /usr/share/nginx/html/`

を行うと、

こんな感じになります。

```
root@ubuntu-bionic:/var/lib/docker/overlay2/3409092e8aeadb6843f75e35deff95df372c40c0ef68a9e5f5d9d2309d004234# tree .
.
├── diff
│   └── usr
│       └── share
│           └── nginx
│               └── html
│                   └── index.html
├── link
├── lower
└── work

6 directories, 3 files
```

このように、docker buildでは必要なレイヤーを作成していきます。

ぞれぞれのレイヤーは特に圧縮などはされていないので、diskの容量を消費していきますね。

レイヤーの配置は下記コマンドでも確認できます。

```
vagrant@ubuntu-bionic:~/build$ sudo docker image inspect test
[
    {
        "Id": "sha256:7989de10cf016bfb5d3975f621161dafb2ea97dd13f997e01071c981999015d9",
        "RepoTags": [
            "test:latest"
        ],
        "RepoDigests": [],
        "Parent": "sha256:6149720787eeb5559a9e69dd65449d491a4aadbff30147e6ffec2a39b4a05e3c",
        "Comment": "",
        "Created": "2020-11-20T22:08:48.512482847Z",
        "Container": "",
        "ContainerConfig": {
            "Hostname": "",
            "Domainname": "",
            "User": "",
            "AttachStdin": false,
            "AttachStdout": false,
            "AttachStderr": false,
            "ExposedPorts": {
                "80/tcp": {}
            },
            "Tty": false,
            "OpenStdin": false,
            "StdinOnce": false,
            "Env": [
                "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin",
                "NGINX_VERSION=1.19.4",
                "NJS_VERSION=0.4.4",
                "PKG_RELEASE=1"
            ],
            "Cmd": [
                "/bin/sh",
                "-c",
                "#(nop) COPY file:d8d86d4e405f41117611f49a970cf2935c8f3d3056b1bd7f6f20639e94d14754 in /usr/share/nginx/html/ "
            ],
            "Image": "sha256:6149720787eeb5559a9e69dd65449d491a4aadbff30147e6ffec2a39b4a05e3c",
            "Volumes": null,
            "WorkingDir": "",
            "Entrypoint": [
                "/docker-entrypoint.sh"
            ],
            "OnBuild": null,
            "Labels": {
                "maintainer": "NGINX Docker Maintainers <docker-maint@nginx.com>"
            },
            "StopSignal": "SIGTERM"
        },
        "DockerVersion": "19.03.13",
        "Author": "",
        "Config": {
            "Hostname": "",
            "Domainname": "",
            "User": "",
            "AttachStdin": false,
            "AttachStdout": false,
            "AttachStderr": false,
            "ExposedPorts": {
                "80/tcp": {}
            },
            "Tty": false,
            "OpenStdin": false,
            "StdinOnce": false,
            "Env": [
                "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin",
                "NGINX_VERSION=1.19.4",
                "NJS_VERSION=0.4.4",
                "PKG_RELEASE=1"
            ],
            "Cmd": [
                "nginx",
                "-g",
                "daemon off;"
            ],
            "Image": "sha256:6149720787eeb5559a9e69dd65449d491a4aadbff30147e6ffec2a39b4a05e3c",
            "Volumes": null,
            "WorkingDir": "",
            "Entrypoint": [
                "/docker-entrypoint.sh"
            ],
            "OnBuild": null,
            "Labels": {
                "maintainer": "NGINX Docker Maintainers <docker-maint@nginx.com>"
            },
            "StopSignal": "SIGTERM"
        },
        "Architecture": "amd64",
        "Os": "linux",
        "Size": 23049084,
        "VirtualSize": 23049084,
        "GraphDriver": {
            "Data": {
                "LowerDir": "/var/lib/docker/overlay2/69f4883c42c734c53d273ffdceedf5d1afcbc303e282b62078e5fd05680ecadb/diff:/var/lib/docker/overlay2/2a7aa612a61f74c50e3b59adde92eec888307e7e6d9316c63416d1e4d966c320/diff:/var/lib/docker/overlay2/4258952a3b61600f79d599b729e75d0ef66d28591ac6c7c241f15f631d683ff6/diff:/var/lib/docker/overlay2/7a089583c5e6240bb9ef515b658a90f69b26fc69b7014133ee8bb40c41054f8e/diff:/var/lib/docker/overlay2/ff97a4397a806cf5fd78891f6e1b27290a1234c6cc815c0eb5f5b7d8d87be8a0/diff:/var/lib/docker/overlay2/7c8911a3eb6cf7c9f9a4aa32aced5ed4ba28bc412eda5fe49ad221f69a82003b/diff:/var/lib/docker/overlay2/8e1e69bd9cdab4e28e2d76333e07b5581b2b07380e7e21b7255c934859e1ef96/diff",
                "MergedDir": "/var/lib/docker/overlay2/971d16a26d208e7f6715bfd1d2f1c3757bf2abfc3e975124128dd02bbb8ead12/merged",
                "UpperDir": "/var/lib/docker/overlay2/971d16a26d208e7f6715bfd1d2f1c3757bf2abfc3e975124128dd02bbb8ead12/diff",
                "WorkDir": "/var/lib/docker/overlay2/971d16a26d208e7f6715bfd1d2f1c3757bf2abfc3e975124128dd02bbb8ead12/work"
            },
            "Name": "overlay2"
        },
        "RootFS": {
            "Type": "layers",
            "Layers": [
                "sha256:ace0eda3e3be35a979cec764a3321b4c7d0b9e4bb3094d20d3ff6782961a8d54",
                "sha256:93e19e6dd56b059a7356aa864f9916185559a56ff18da4b189a3fcc13d3aa0ee",
                "sha256:e2a648dc6400feb0d24484d83ba5800ed58adbcfdb9986f80d4ae106ac891968",
                "sha256:2c8583333eb335ac202e1b798f9078ad0128f34c25a3dfe6f05e6816d3480900",
                "sha256:2367050c34dd9fd39052c35bf9a930e94a2be7202b33e1f8e64c1f04dd82011e",
                "sha256:c2ec8348d1b3d91386c85a41b64fdf047978a241ee2084f0f1ef754908488f76",
                "sha256:adf98b9b13ed145fa045990053871348de49fd264888d2590143aa627a503bfd",
                "sha256:78040b1cb1aa02b7412fe041e1437f9bd5394389c24e0c0a9ec98f4d53ea539f"
            ]
        },
        "Metadata": {
            "LastTagTime": "2020-11-20T22:08:48.569617179Z"
        }
    }
]
# こうもできます
vagrant@ubuntu-bionic:~/build$ sudo docker image inspect test -f "{{json .GraphDriver.Data}}" | jq
{
  "LowerDir": "/var/lib/docker/overlay2/69f4883c42c734c53d273ffdceedf5d1afcbc303e282b62078e5fd05680ecadb/diff:/var/lib/docker/overlay2/2a7aa612a61f74c50e3b59adde92eec888307e7e6d9316c63416d1e4d966c320/diff:/var/lib/docker/overlay2/4258952a3b61600f79d599b729e75d0ef66d28591ac6c7c241f15f631d683ff6/diff:/var/lib/docker/overlay2/7a089583c5e6240bb9ef515b658a90f69b26fc69b7014133ee8bb40c41054f8e/diff:/var/lib/docker/overlay2/ff97a4397a806cf5fd78891f6e1b27290a1234c6cc815c0eb5f5b7d8d87be8a0/diff:/var/lib/docker/overlay2/7c8911a3eb6cf7c9f9a4aa32aced5ed4ba28bc412eda5fe49ad221f69a82003b/diff:/var/lib/docker/overlay2/8e1e69bd9cdab4e28e2d76333e07b5581b2b07380e7e21b7255c934859e1ef96/diff",
  "MergedDir": "/var/lib/docker/overlay2/971d16a26d208e7f6715bfd1d2f1c3757bf2abfc3e975124128dd02bbb8ead12/merged",
  "UpperDir": "/var/lib/docker/overlay2/971d16a26d208e7f6715bfd1d2f1c3757bf2abfc3e975124128dd02bbb8ead12/diff",
  "WorkDir": "/var/lib/docker/overlay2/971d16a26d208e7f6715bfd1d2f1c3757bf2abfc3e975124128dd02bbb8ead12/work"
}
```


## RUNでコマンドを走らせた場合(まだ確認中)
RUNでコマンドを走らせる場合は、ちょっと様子が異なります。

実際にコマンドを走らせてレイヤーを保存するためです。

そのため、予めもととなるレイヤーを合わせてコマンドを実行していると思われます。(多分。。。)

https://gist.github.com/SpringMT/856715ed8e66f9665c42c2f1c73e45b6

clone

```
21:17:05.410116 clone(child_stack=NULL, flags=CLONE_VM|CLONE_VFORK|SIGCHLD) = 16384
```
```
21:17:05.524898 write(19, "33\r\n{\"stream\":\"Step 2/4 : RUN apk add --no-cache jq\"}\r\n\r\n", 57) = 57
```


init mount
```
21:17:05.667754 mount("overlay", "/var/lib/docker/overlay2/4294e9f6db70b0818156f99a122c437886f6024efde21c3c5f71d1a0d051b4d6-init/merged", "overlay", 0, "index=off,lowerdir=/var/lib/docker/overlay2/l/PQZNLDA5HCYZC7DGQKCMIXBWUL:/var/lib/docker/overlay2/l/H666Q727TFO3FBFQR3SJH2LKCG:/var/lib/docker/overlay2/l/3YFNEUFUG56R7PC652LZO3HAJQ:/var/lib/docker/overlay2/l/SZZPM4XTPEDLKIUMMCCFAMYP2M:/var/lib/docker/overlay2/l/377KU4VZQXCWQBD7A32YNJ2YQ2,upperdir=/var/lib/docker/overlay2/4294e9f6db70b0818156f99a122c437886f6024efde21c3c5f71d1a0d051b4d6-init/diff,workdir=/var/lib/docker/overlay2/4294e9f6db70b0818156f99a122c437886f6024efde21c3c5f71d1a0d051b4d6-init/work") = 0
```
多分ここらへんなはず。

https://github.com/moby/moby/blob/46cdcd206c56172b95ba5c77b827a722dab426c5/layer/layer_store.go#L681

https://github.com/moby/moby/blob/ffd0861b8b5181c90dad4a2b13ad042d26d8996a/daemon/graphdriver/overlay2/overlay.go

```
21:17:05.705012 umount2("/var/lib/docker/overlay2/4294e9f6db70b0818156f99a122c437886f6024efde21c3c5f71d1a0d051b4d6-init/merged", MNT_DETACH) = 0	
```
```
21:17:05.795560 mount("overlay", "/var/lib/docker/overlay2/4294e9f6db70b0818156f99a122c437886f6024efde21c3c5f71d1a0d051b4d6/merged", "overlay", 0, "index=off,lowerdir=/var/lib/docker/overlay2/l/QGZPGB4RCLY5U5VHMBUKR4LNBH:/var/lib/docker/overlay2/l/PQZNLDA5HCYZC7DGQKCMIXBWUL:/var/lib/docker/overlay2/l/H666Q727TFO3FBFQR3SJH2LKCG:/var/lib/docker/overlay2/l/3YFNEUFUG56R7PC652LZO3HAJQ:/var/lib/docker/overlay2/l/SZZPM4XTPEDLKIUMMCCFAMYP2M:/var/lib/docker/overlay2/l/377KU4VZQXCWQBD7A32YNJ2YQ2,upperdir=/var/lib/docker/overlay2/4294e9f6db70b0818156f99a122c437886f6024efde21c3c5f71d1a0d051b4d6/diff,workdir=/var/lib/docker/overlay2/4294e9f6db70b0818156f99a122c437886f6024efde21c3c5f71d1a0d051b4d6/work") = 0
```
```
21:17:05.801765 umount2("/var/lib/docker/overlay2/4294e9f6db70b0818156f99a122c437886f6024efde21c3c5f71d1a0d051b4d6/merged", MNT_DETACH) = 0
```
```
21:17:05.902354 mount("overlay", "/var/lib/docker/overlay2/4294e9f6db70b0818156f99a122c437886f6024efde21c3c5f71d1a0d051b4d6/merged", "overlay", 0, "index=off,lowerdir=/var/lib/docker/overlay2/l/QGZPGB4RCLY5U5VHMBUKR4LNBH:/var/lib/docker/overlay2/l/PQZNLDA5HCYZC7DGQKCMIXBWUL:/var/lib/docker/overlay2/l/H666Q727TFO3FBFQR3SJH2LKCG:/var/lib/docker/overlay2/l/3YFNEUFUG56R7PC652LZO3HAJQ:/var/lib/docker/overlay2/l/SZZPM4XTPEDLKIUMMCCFAMYP2M:/var/lib/docker/overlay2/l/377KU4VZQXCWQBD7A32YNJ2YQ2,upperdir=/var/lib/docker/overlay2/4294e9f6db70b0818156f99a122c437886f6024efde21c3c5f71d1a0d051b4d6/diff,workdir=/var/lib/docker/overlay2/4294e9f6db70b0818156f99a122c437886f6024efde21c3c5f71d1a0d051b4d6/work") = 0
```

```
21:17:05.910335 setns(11, CLONE_NEWNET) = 0
```

最終的には他の命令と同様にレイヤーが作成されて完了となります。

## 参考資料
* https://docs.docker.com/storage/storagedriver/#images-and-layers
* https://docs.docker.com/storage/storagedriver/overlayfs-driver/#how-container-reads-and-writes-work-with-overlay-or-overlay2
* https://www.creationline.com/lab/35518
