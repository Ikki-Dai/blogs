---
singleServerConfig:
  address: redis://tudc-account-test.sclla2.ng.0001.apse1.cache.amazonaws.com:6379
  database: 2
  connectionMinimumIdleSize: 8
  connectionPoolSize: 16
#  idleConnectionTimeout: 30000
  keepAlive: true

threads: 8
nettyThreads: 8
codec: !<org.redisson.codec.MarshallingCodec> { }
transportMode: "NIO"


```java

        BlockingQueue<String> queue;
        if (Boolean.TRUE.equals(localCachedMap.get(clientId))) {
            queue = redisson.getBlockingQueue("SSE_QUE:" + clientId);
        } else {
            return new ResponseEntity<>(HttpStatus.GONE);
        }

        try {
            queue.offer(objectMapper.writeValueAsString(scanRequest));
        } catch (JsonProcessingException e) {
            e.printStackTrace();
        }

        return ResponseEntity.noContent().build();

```

```java
    @GetMapping("/qr-code/{client_id}")
    public ResponseEntity<SseEmitter> QRLogin(@PathVariable("client_id") String clientId) {
        log.info("request in....");
        String authcode = UUID.randomUUID().toString();
        SseEmitter sseEmitter = new SseEmitter(300 * 1000L);
        String id = clientId + '@' + authcode;
        BlockingQueue<String> queue = redisson.getBlockingQueue("SSE_QUE:" + id);
        localCachedMap.put(id, true);
        sseEmitter.onCompletion(() -> {
            localCachedMap.remove(id);
            queue.clear();
            log.info("sse {} completed", id);
        });


        threadPoolTaskExecutor.submit(() -> {
            Map<String, String> body = new HashMap<>();
            body.put("clientId", clientId);
            body.put("authCode", authcode);
            body.put("redirectUrl", "immigrant.com/login/qr");

            try {
                sseEmitter.send(body, MediaType.APPLICATION_JSON);
                while (true) {
                    String s = queue.take();
                    log.info(s);
                    if ("END".equals(s)) {
                        sseEmitter.complete();
                    } else {
                        sseEmitter.send(s);
                    }
                }

            } catch (InterruptedException | IOException e) {
                e.printStackTrace();
            }


        });

        return ResponseEntity.ok(sseEmitter);
```