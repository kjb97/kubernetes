# Error: UPGRADE FAILED: another operation (install/upgrade/rollback) is in progress
 - 릴리즈가 upgrade 또는 install 중 오류가 발생해 pending 상태에 빠진 상태에서 upgrade나 install할 경우 발생하는 error
- 대부분 rollback을 진행하면 해결됨
``` helm rollback <릴리즈> -n <namespace>```

- 그런데 가끔씩 아예 list에 없는 경우가 있음
   그럴 땐 namespace에서 뽑아보면 나올 수 있음
 ``` helm ls -a -n <namespace> ```
