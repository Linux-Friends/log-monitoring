
<img src="https://capsule-render.vercel.app/api?type=waving&color=FFE146&height=300&section=header&text=Monitoring&fontSize=70&fontColor=FFFFFF&animation=fadeIn&width=1200" width="1200" />


<br>


<br>

## 📍 Contents
- [1️⃣ Contributors](#1%EF%B8%8F⃣-contributors)
- [2️⃣ Overview](#2%EF%B8%8F⃣-overview)
- [3️⃣ Expectations](#3%EF%B8%8F⃣-expectations)
- [4️⃣ Skills](#4%EF%B8%8F⃣-skills)
- [5️⃣ How to do](#5%EF%B8%8F⃣-how-to-do)
- [6️⃣ Trouble Shooting](#6%EF%B8%8F%E2%83%A3-trouble-shooting)
- [7️⃣ Retrospective](#7%EF%B8%8F%E2%83%A3-retrospective)

<br>
<br>

## 1️⃣ Contributors
<br>

|<img src="https://github.com/DoomchitYJ.png" width="220" />|<img src="https://github.com/imhaeunim.png" width="220" />|<img src="https://github.com/jinhyunpark929.png" width="220" />|<img src="https://github.com/letmeloveyou82.png" width="220" />|
|:-:|:-:|:-:|:-:|
|박영진<br/>[@DoomchitYJ](https://github.com/DoomchitYJ)|임하은<br/>[@imhaeunim](https://github.com/imhaeunim)|박진현<br/>[@jinhyunpark929](https://github.com/jinhyunpark929)|최윤정<br/>[@letmeloveyou82](https://github.com/letmeloveyou82)|

<br>

## 2️⃣ Overview

### ⚽ 목표 <br>

- MySQL DB 서버의 메모리를 모니터링하고 로그 저장
- 메모리 사용량에 따라 조치 자동 실행
<br>


## 3️⃣ Expectations
### 🚨 조치 <br>

- **메모리 70% 초과 시 경고 메일만 발송하여 사전 대응**
- **메모리 90% 초과 시 DB 서비스는 유지하면서도 부하를 줄일 수 있는 조치 실행**
    - 오래된 **쿼리 종료** (실행 시간이 긴 쿼리 정리)
    - **Idle 세션 종료** (불필요한 연결 해제)
    - **캐시 정리** (OS 레벨 캐시 해제)
    - **스왑 해제 & 재활성화** (Swap 사용률이 높다면)
<br>

## 4️⃣ Skills

### 🛠 Skills

**Tech Stack**

<div>
<img src="https://img.shields.io/badge/linux-FCC624?style=for-the-badge&logo=linux&logoColor=black">
<img src="https://img.shields.io/badge/mysql-4479A1?style=for-the-badge&logo=mysql&logoColor=white">
<img src="https://img.shields.io/badge/jmeter-D22128?style=for-the-badge&logo=apachejmeter&logoColor=white">

</div>

<br>

**Co-work Stack**

<div>
<img src="https://img.shields.io/badge/notion-000000?style=for-the-badge&logo=notion&logoColor=white">
<img src="https://img.shields.io/badge/github-303a50?style=for-the-badge&logo=github&logoColor=white">
<img src="https://img.shields.io/badge/slack-e01e5a?style=for-the-badge&logo=slack&logoColor=white">
</div>

<br>


## 5️⃣ How to do
### 1. 쉘 스크립트 작성 및 저장

#### ✏️ 쉘 스크립트
```bash
#!/bin/bash

# 설정값
EMAIL="admin@example.com"   # 관리자 이메일
MEM_THRESHOLD=70            # 경고 알림 기준 (%)
CRITICAL_THRESHOLD=90       # 강제 조치 기준 (%)
LOG_FILE="/var/log/db_memory_monitor.log"
HOSTNAME=$(hostname)
MYSQL_USER="root"           # MySQL 사용자
MYSQL_PASSWORD="your_password" # MySQL 비밀번호

# 현재 메모리 사용량 확인
MEMORY_USAGE=$(free | awk '/Mem:/ {printf("%.0f"), $3/$2 * 100}')
DATE=$(date "+%Y-%m-%d %H:%M:%S")

# 로그 기록 함수
log_message() {
    echo "$DATE $1" | tee -a $LOG_FILE
}

# 70% 초과 시 관리자에게 메일 알림
if [ "$MEMORY_USAGE" -ge "$MEM_THRESHOLD" ]; then
    log_message "[ALERT] Memory Usage High: $MEMORY_USAGE% - Sending alert email"

    MESSAGE="🚨 [ALERT] DB Server ($HOSTNAME)\nMemory Usage: $MEMORY_USAGE%\nPlease check the server!"
    echo -e "$MESSAGE" | mail -s "[ALERT] DB Server High Memory Usage" $EMAIL
fi

# 90% 초과 시 긴급 조치 실행
if [ "$MEMORY_USAGE" -ge "$CRITICAL_THRESHOLD" ]; then
    log_message "[CRITICAL] Memory Usage Exceeded 90%! Taking actions..."

    # 오래 실행 중인 쿼리 종료 (10초 이상 실행된 쿼리)
    log_message "[INFO] Killing long running queries..."
    mysql -u "$MYSQL_USER" -p"$MYSQL_PASSWORD" -e "
        SELECT id FROM information_schema.processlist WHERE time > 10;" | while read id; do
        mysql -u "$MYSQL_USER" -p"$MYSQL_PASSWORD" -e "KILL $id;"
    done

    # Idle 세션 종료 (비활성 세션 제거)
    log_message "[INFO] Terminating idle connections..."
    mysql -u "$MYSQL_USER" -p"$MYSQL_PASSWORD" -e "
        SELECT id FROM information_schema.processlist WHERE command='Sleep' AND time > 60;" | while read id; do
        mysql -u "$MYSQL_USER" -p"$MYSQL_PASSWORD" -e "KILL $id;"
    done

    # 캐시 정리 (리눅스 OS 레벨 캐시 정리)
    log_message "[INFO] Dropping caches to free memory..."
    sync; echo 3 | sudo tee /proc/sys/vm/drop_caches

    # Swap 사용량 확인 후 필요하면 해제 및 재활성화
    SWAP_USED=$(free | awk '/Swap:/ {print $3}')
    if [ "$SWAP_USED" -gt 512 ]; then
        log_message "[INFO] Swap usage detected ($SWAP_USED MB). Restarting swap..."
        sudo swapoff -a && sudo swapon -a
    fi

    # 관리자에게 조치 완료 메일 전송
    log_message "[INFO] Actions completed. Sending report to admin..."
    MESSAGE="🔥 [CRITICAL] DB Server ($HOSTNAME)\nMemory Usage: $MEMORY_USAGE%\nActions Taken:\n- Long running queries killed\n- Idle connections terminated\n- Cache cleared\n- Swap reset (if needed)"
    echo -e "$MESSAGE" | mail -s "[CRITICAL] DB Server Memory Alert - Actions Taken" $EMAIL
fi

```

#### ✉ 스크립트 저장
```bash
nano /path/to/db_memory_monitor.sh
```

<br>

### 2. 권한 부여
#### 🖐 파일에 실행 권한 부여
```bash
chmod +x /path/to/db_memory_monitor.sh
```

<br>

### 3. 자동 실행
#### 🕐 5분마다 자동 실행되도록 cron에 등록
```bash
crontab -e
*/5 * * * * /path/to/db_memory_monitor.sh
```

<br>


## 6️⃣ Trouble Shooting



**🧨원인:**


**👌 해결방안**


<br>

## 7️⃣ Retrospective

