# intro
- linux account password 가 만료됐다는 메세지가 나왔고, 어쩔수 없이 다른 비밀번호로 변경 후, 이전 비밀번호로 돌려놓고 싶었음 
- 비밀번호 유효 기간을 없애고싶었음 

# howto 
## change password

- account 지정해서 변경하는 경우
```bash
$ sudo passwd fransoaardi

Changing password for user fransoaardi.
New password: (새로운 패스워드 입력)
Retype new password: (새로운 패스워드 재입력)
passwd: all authentication tokens updated successfully.
```

- 본인의 password 변경하는경우
```bash
$ passwd

Changing password for user fransoaardi.
New password: (새로운 패스워드 입력)
Retype new password: (새로운 패스워드 재입력)
passwd: all authentication tokens updated successfully.
```

## eliminating expiry date 

```bash
# 계정의 비밀번호 상태를 볼 수 있음 
$ chage -l fransoaardi

마지막으로 암호를 바꾼 날					:11월 26, 2019
암호 만료					: 2월 25, 2020
암호가 비활성화 기간					:안함
계정 만료						:안함
암호를 바꿀 수 있는 최소 날 수		: 0
암호를 바꿔야 하는 최대 날 수		: 91
암호 만료 예고를 하는 날 수		: 7

$ sudo chage -M 99999 fransoaardi

마지막으로 암호를 바꾼 날					:11월 26, 2019
암호 만료					:안함
암호가 비활성화 기간					:안함
계정 만료						:안함
암호를 바꿀 수 있는 최소 날 수		: 0
암호를 바꿔야 하는 최대 날 수		: 99999
```

# reference
- https://www.linux.co.kr/home/lecture/index.php?cateNo=&secNo=&theNo=&leccode=247
- https://www.cyberciti.biz/tips/setting-off-password-aging-expiration.html
