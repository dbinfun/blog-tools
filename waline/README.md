[Waline官网地址](https://waline.js.org/)

使用直接[运行部署](https://waline.js.org/guide/deploy/vps.html#%E7%9B%B4%E6%8E%A5%E8%BF%90%E8%A1%8C-%E6%8E%A8%E8%8D%90)比较方便

## step.0

安装`nodejs`,略...

## step.1

创建一个文件夹存放项目，并下载`waline`依赖

```sh
mkdir waline
cd waline
npm install @waline/vercel
```

## step.2

创建启动脚本等

`start.sh`:

```sh
##### 设置环境变量 #####
# 生产环境
export NODE_ENV=production
# SQLite 数据库文件的路径，该路径不包含文件名本身
export SQLITE_PATH=$(pwd)
# 用户登录密钥，随机字符串即可
export JWT_TOKEN=*****
# 藏评论者的归属地
export DISABLE_REGION=true
# 下面是一些信息
export SITE_NAME=dbin的小站
export SITE_URL=https://dbinfun.net
# 安全域名
export SECURE_DOMAINS=example.com
# 必须登录才能评论,可以不要
export LOGIN=force
# 邮箱服务
export SMTP_SERVICE=Gmail
export SMTP_USER=***@gmail.com
# 邮箱密码,gmail需要开启两步验证后获取独立密码
export SMTP_PASS=****
export SMTP_SECURE=true
export SENDER_NAME=dbin
export SENDER_EMAIL=***@gmail.com
##### 运行waline #####
node node_modules/@waline/vercel/vanilla.js
```

`stop.sh`

```sh
PID=$(ps -ef | grep node_modules/@waline/vercel/vanilla.js | grep -v grep | awk '{ print $2 }')
if [ -z "$PID" ]
then
        echo Application is already stopped
else
        echo kill -9 $PID
        kill -9 $PID
fi
```

`begin.sh`

```sh
command="./start.sh"

# 日志文件路径
log_file="log.log"

# 使用 nohup 运行命令，并将输出重定向到日志文件
nohup $command > $log_file 2>&1 &
```

## step.3

运行`begin.sh`

```shell
chmod 755 begin.sh
chmod 755 start.sh
chmod 755 stop.sh
./begin.sh
```

## other

我在`valaxy`博客中重写了组件，方便自定表情

再`components`目录下创建`WalineClient.vue`

```vue
<script lang="ts" setup>
import { isDark } from 'valaxy'
import { computed, onMounted } from 'vue'
import { useI18n } from 'vue-i18n'
import { useRoute } from 'vue-router'
// @ts-expect-error vue waline component type
import { Waline } from '@waline/client/dist/component'
import { commentCount, pageviewCount } from '@waline/client'

import '@waline/client/dist/waline.css'
import type { WalineOptions } from 'valaxy-addon-waline/types'

const props = defineProps<{
  options: WalineOptions
}>()

const route = useRoute()
const { locale } = useI18n()
const path = computed(() => props.options.path || route.path.replace(/\/$/, ''))
// 自定一emoji
const emoji = computed(() => function (){
  let emoji = props.options.emoji
  if(emoji===undefined){
    return [
      'https://unpkg.com/@waline/emojis@1.1.0/bilibili/',
      'https://unpkg.com/@waline/emojis@1.1.0/bmoji/',
      'https://unpkg.com/@waline/emojis@1.1.0/qq/'
    ]
  }else{
    return emoji
  }
}())

onMounted(() => {
  console.log("tests")
  const { pageview, comment } = props.options
  if (pageview) {
    pageviewCount({
      serverURL: props.options.serverURL,
      path: path.value,
      selector: typeof pageview === 'string' ? pageview : undefined,
    })
  }

  if (comment) {
    commentCount({
      serverURL: props.options.serverURL,
      path: path.value,
      selector: typeof comment === 'string' ? comment : undefined,
    })
  }
})
</script>

<template>
  <Waline v-bind="options" :server-u-r-l="options.serverURL" :lang="locale" :path="path" :dark="isDark" :emoji="emoji" />
</template>

<style>
:root {
  --waline-theme-color: var(--va-c-primary);
  --waline-active-color: var(--va-c-primary-light);
}
</style>
```

在`valaxy.config.ts`中添加

```ts
addons: [
  addonWaline({
    serverURL: "https://comments.dbinfun.net/",
    reaction: true,
    emoji: [
      'https://unpkg.com/@waline/emojis@1.1.0/bilibili/',
      'https://unpkg.com/@waline/emojis@1.1.0/bmoji/',
      'https://unpkg.com/@waline/emojis@1.1.0/qq/',
      'https://unpkg.com/@waline/emojis@1.1.0/tieba/',
      'https://unpkg.com/@waline/emojis@1.1.0/tw-emoji/',
      'https://unpkg.com/@waline/emojis@1.1.0/weibo/',
    ]
  }),
],
```

