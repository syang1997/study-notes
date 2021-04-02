# 传输

## 传输API

传输API的核心是Channel，被用于所有的io操作。每一个Channel都会被分配一个ChannelPipeline和ChannelConfig。ChannelConfig包含了Channel的所有配置，支持热更新。