# CI image for Apache Flink

## Build

**Never re-use a tag because it might break CI for older Flink commits.**

```
docker build -t apache/flink-ci:<tag> base
docker push apache/flink-ci:<tag>
```
