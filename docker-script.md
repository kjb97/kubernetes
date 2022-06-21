# Docker push script

- 1. tar 파일을 load
- 2. tag 지정
- 3. 레지스트리로 push
- 4. tag 이전 이미지 삭제
```
#! /bin/bash

reg="10.250.xx.xx:5000"
dir="$1"

for f in $dir/*.tar; do
  image_name=$(cat $f | docker load | awk '{print $3}')
  docker tag $image_name $reg/$image_name
  docker push $reg/$image_name
  docker rmi $image_name
done
```
- 실행 시에 directory 경로를 넘겨야 함
```
sh docker.sh /home/centos/
```
