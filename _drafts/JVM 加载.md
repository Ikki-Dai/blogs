```mermaid
graph  TB

start(开始) --> load{是否装载?}

style load fill: pink
style classLoader fill: pink

subgraph 加载
  load --N--> classLoader{装载顺利}
end

style link fill: green
style init fill: green
style call fill: green

load -- Y --> link[连接]
link --> init[初始化类]
init --> call[调用类main方法]

call --> stop(结束)

style exception fill: red

classLoader --N--> exception[抛出异常]
classLoader --Y--> link
```