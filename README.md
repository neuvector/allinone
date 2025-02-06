# allinone

## How to build

Prepare below images with the same tag, say `latest`.
- xxx/controller:latest
- xxx/enforcer:latest
- xxx/manager:latest

Run the below command to repack them as NV's allinone image.
```
docker build -f package/Dockerfile \
    --build-arg NV_VERSION=latest --build-arg SRCREPO=xxx \
    -t "xxx/allinone:latest" --pull .
```
