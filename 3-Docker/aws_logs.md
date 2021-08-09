AWSでfirelensの設定をすると、sidecarで下記イメージがsidecarで入る

https://github.com/aws/aws-for-fluent-bit

## aws-ecs-agent( https://github.com/aws/amazon-ecs-agent )による設定
多分ここらへん

https://github.com/aws/amazon-ecs-agent/blob/225bc3a556bd2d1759ab27b23f54e7e68086c9f0/agent/taskresource/firelens/firelens_unix.go#L470

https://github.com/aws/amazon-ecs-agent/blob/225bc3a556bd2d1759ab27b23f54e7e68086c9f0/agent/taskresource/firelens/firelensconfig_unix.go#L118

実際には下記moduleがtemplateとかを持っている

https://github.com/awslabs/go-config-generator-for-fluentd-and-fluentbit

## fluent-bitのバッファリング
基本的には時間でflushする

https://docs.fluentbit.io/manual/administration/configuring-fluent-bit/configuration-file

それ以外に、output pluginとかがバッファリングする

## 参考
* https://yasuharu519.hatenablog.com/entry/2019/12/19/142454
* https://qiita.com/flatnyat/items/3cad79fc5a323ae3245c
* https://qiita.com/orfx/items/6d0709d1a175793cad03
* https://github.com/aws/aws-for-fluent-bit/tree/mainline/ecs

* https://github.com/aws/aws-for-fluent-bit/issues/104 
