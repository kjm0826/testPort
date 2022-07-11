# 22년 7월 11일 SMS 전환 시나리오

<br />

### 사전체크사항 : GS ITM AGENT -> 운영DB 방화벽 체크 

<br />

>   1번 DB 체크

```javascript
(cleantopia-talk-prd-node40) root ~ $ telnet 192.168.1.161 1521
Trying 192.168.1.161...
Connected to 192.168.1.161.
Escape character is '^]'.
```

>   2번 DB 체크

```javascript
(cleantopia-talk-prd-node40) root ~ $ telnet 192.168.1.163 1521
Trying 192.168.1.163...
Connected to 192.168.1.163.
Escape character is '^]'.
```


<br />

### 1. SMS AGENT 전환 [ 포유링크 -> GS ITM ]

<br />
  
>  [AS-IS] SMS AGENT 중지 (포유링크 서비스)  
>  - 서버정보 : 192.168.2.205  
>  - 계정정보 :  / 

<br />

```javascript
edu:/] ps -ef | egrep '(Agent|Mon|bizmt)'
edu:/] kill -9 $(ps -ef | egrep '(Agent|Mon|bizmt)' | awk '{print $2}')
```

<br />
  
>  [TO-BE] SMS AGENT 기동 (GS ITM 서비스)  
>  - 서버정보 : 192.168.2.205  
>  - 계정정보 :  / 
> 
<br />

>  대량문자 AGENT 기동 
```javascript
(cleantopia-talk-prd-node40) root ~ $ cd /home/was/GBMAgent_mass
(cleantopia-talk-prd-node40) root ~ $ sh start.sh
```

>  대량문자 AGENT 종료  
```javascript
(cleantopia-talk-prd-node40) root ~ $ cd /home/was/GBMAgent_mass
(cleantopia-talk-prd-node40) root ~ $ sh stop.sh
```

>  대량문자 로그 경로 
```javascript
(cleantopia-talk-prd-node40) root ~ $ cd /home/was/GBMAgent_mass/log
```

>  실시간 AGENT 기동 
```javascript
(cleantopia-talk-prd-node40) root ~ $ cd /home/was/GBMAgent_small
(cleantopia-talk-prd-node40) root ~ $ sh start.sh
```

>  실시간 AGENT 종료  
```javascript
(cleantopia-talk-prd-node40) root ~ $ cd /home/was/GBMAgent_mass
(cleantopia-talk-prd-node40) root ~ $ sh stop.sh
```

>  실시간 로그 경로 
```javascript
(cleantopia-talk-prd-node40) root ~ $ cd /home/was/GBMAgent_small/log
```

<br />

### 2. 기존 테이블 RENAME 처리 테이블명_OLD  [ 1~7월 ]
1. ADG에 영향이 있을 가능성이 존재하여 1~7월 테이블명만 RENAME 처리  
   ( 6개월 데이터 마이그레이션시 3천만건 가량으로 부하가능성 존재 )

2. 과거 SMS 내역 조회 클레임 발생 시 1~2개월 식 데이터 마이그레이션 처리 
   
<br />

```SQL
ALTER TABLE SDK_SMS_REPORT_2207 RENAME TO SDK_SMS_REPORT_2207_OLD;  
ALTER TABLE SDK_MMS_REPORT_2207 RENAME TO SDK_MMS_REPORT_2207_OLD;  
ALTER TABLE SDK_SMS_REPORT_DETAIL_2207 RENAME TO SDK_SMS_REPORT_DETAIL_2207_OLD;  
ALTER TABLE SDK_MMS_REPORT_DETAIL_2207 RENAME TO SDK_MMS_REPORT_DETAIL_2207_OLD;
```

<br />

###  3. 신규테이블로 데이터 마이그레이션 [ 6~7월 2개월 데이터 800 만여건 ]
1. SMSUSER/LGEVENTUSER 각각 처리 

<br />

> SMS 테이블
> 
```sql
INSERT INTO SDK_SMS_REPORT_DETAIL_2207
SELECT A.MSG_ID,
       A.USER_ID,
       A.JOB_ID,
       B.SUBJOB_ID,
       A.SCHEDULE_TYPE,
       A.SUBJECT,
       A.SMS_MSG,
       A.CALLBACK_URL,
       A.NOW_DATE,
       A.SEND_DATE,
       CALLBACK,
       DEST_TYPE,
       DEST_COUNT,
       DEST_INFO,
       KT_OFFICE_CODE,
       CDR_ID,
       DEST_NAME,
       PHONE_NUMBER,
       RESULT,
       TCS_RESULT,
       FEE,
       DELIVER_DATE,
       REPORT_RES_DATE,
       MSG_SUBJOB_TYPE,
       STAT_STATUS,
       TELCOINFO,
       STATUS_TEXT,
       RESERVED1,
       RESERVED2,
       RESERVED3,
       RESERVED4,
       RESERVED5,
       RESERVED6,
       SUCC_COUNT,
       FAIL_COUNT,
       CANCEL_STATUS,
       CANCEL_COUNT,
       CANCEL_REQ_DATE,
       CANCEL_RESULT,
       DELIVER_STATUS,
       DELIVER_COUNT,
       DELIVER_REQ_DATE,
       DELIVER_RESULT,
       STD_ID,
       RESERVED7,
       RESERVED8,
       RESERVED9,
       C.SEND_MESSAGE KKO_MSG,
       '' SEND_SPID
FROM  SDK_SMS_REPORT_2207_OLD A
INNER JOIN SDK_SMS_REPORT_DETAIL_2207_OLD B ON A.MSG_ID = B.MSG_ID
LEFT OUTER JOIN TSMS_AGENT_MESSAGE_LOG_202207 C ON A.MSG_ID = C.MSG_ID;
```

> MMS 테이블 
> 
```sql
INSERT INTO SDK_MMS_REPORT_DETAIL_2207
SELECT A.MSG_ID,
       A.USER_ID,
       A.JOB_ID,
       B.SUBJOB_ID,
       A.SCHEDULE_TYPE,
       A.SUBJECT,
       A.NOW_DATE,
       A.SEND_DATE,
       A.CALLBACK,
       A.MMS_MSG,
       CONTENT_COUNT,
       CONTENT_DATA,
       DEST_COUNT,
       DEST_INFO,
       KT_OFFICE_CODE,
       CDR_ID,
       DEST_NAME,
       PHONE_NUMBER,
       RESULT,
       TCS_RESULT,
       DELIVER_DATE,
       READ_TIME,
       MOBILE_INFO,
       FEE,
       REPORT_RES_DATE,
       STAT_STATUS,
       STIME,
       RTIME,
       STATUS_TEXT,
       RESERVED1,
       RESERVED2,
       RESERVED3,
       RESERVED4,
       RESERVED5,
       RESERVED6,
       SUCC_COUNT,
       FAIL_COUNT,
       A.MSG_TYPE,
       STD_ID,
       RESERVED7,
       RESERVED8,
       RESERVED9,
       C.SEND_MESSAGE KKO_MSG,
       SEND_SPID
FROM  SDK_MMS_REPORT_2207_OLD A
INNER JOIN SDK_MMS_REPORT_DETAIL_2207_OLD B ON A.MSG_ID = B.MSG_ID
LEFT OUTER JOIN TSMS_AGENT_MESSAGE_LOG_202207 C ON A.MSG_ID = C.MSG_ID;
```

<br />

> 마이그레이션 후 ADG 모니터링 결과 이상없음

<br />

```javascript
[cleanadg1-ORAADG1]:#oracle#/u01/ugens> sh ./100_while_adg_check_session.sh

Session altered.


SOURCE_DBID NAME                             VALUE                                                            TIME_COMPUTED                  DATUM_TIME
----------- -------------------------------- ---------------------------------------------------------------- ------------------------------ ------------------------------
          0 transport lag                    +00 00:00:00                                                     07/12/2022 01:55:37            07/12/2022 01:55:37
          0 apply lag                        +00 00:00:00                                                     07/12/2022 01:55:37            07/12/2022 01:55:37
          0 apply finish time                +00 00:00:00.000                                                 07/12/2022 01:55:37
          0 estimated startup time           22                                                               07/12/2022 01:55:37


NAME         USABLE_FILE_GB    TOTAL_GB     FREE_GB   usgae(%) USABLE_CALC_GB STATE       TYPE   GROUP_NUMBER
------------ -------------- ----------- ----------- ---------- -------------- ----------- ------ ------------
DATA               5,422.29   17,890.00   11,739.07         34      10,844.57 CONNECTED   NORMAL            1
RECO               1,829.21    4,465.00    3,881.68         13       3,658.43 CONNECTED   NORMAL            2
REDO                 186.06      745.00      744.43          0         558.18 MOUNTED     HIGH              3
```

<br />

### 4. 본사자체발송 대량문자 [TO-BE] 환경 발송 테스트 

<br />

```sql
INSERT INTO TB_AGENT_MESSAGE
 (
        MSG_SEQ,
        SERV_NO,
        TALK_MSG,
        MSG_TITLE,
        MSG_CONT,
        MSG_TYPE,
        BACK_TYPE,
        RECV_NO,
        CALL_NO,
        PROC_TYPE,
        REQ_SEND_DATE,
        TMPL_CODE,
        AGENT_SEND_FLAG,
        REGR_DM,
        REGR_ID,
        IMG_PROC_FLAG
 )
 SELECT  TB_AGENT_MSG_SEQ.NEXTVAL,		-- 메시지 번호 
        '202206101708327',				-- 서비스 코드 [고정값]
        '',								-- 앱 Msg (알림톡/친구톡 등) 내용
        '[IT전략팀] 대량문자 테스트',	    -- LMS/MMS 제목
        '[USTRA 실시간] 문자 테스트 입니다.',	-- 문자(SMS/LMS/MMS) 메시지
        'MS01',  				    		-- MS01(단문) / MS02(LMS)
        '',		 -- 대체처리방법 [ SMS/LMS/MMS : MS01/MS02/MS03, 대체발송없음 : 빈값 ]
        '01033576026' AS ds_hanp,			-- 수신번호
        '0317373430',						-- 발신번호
        'R001',								-- R001(실시간) / M001(대량) 
        SYSDATE,							-- 발송시각(미래시간인 경우 예약발송) 
        '',									-- 카카오 알림(친구)톡 템플릿번호
        '',									-- 발송상태 [B:전송중 E:트레이스, P:발송중]
        SYSDATE,							-- 등록일시
        'TEST-SYS',							-- 등록자
        'N'									-- 이미지 서버 전송 일시
 FROM cleantopiauser.EVENT_MMS_TEMP;
```
<br />

### 5. 알림톡 발송 테스트 

<br />

>  [AS-IS] 단문 알림톡 [TO-BE] 환경 발송 테스트 

```sql
INSERT INTO SMSUSER.SDK_SMS_SEND
(
    MSG_ID,
    USER_ID,
    RESERVED1,
    RESERVED2,
    RESERVED5,
    SCHEDULE_TYPE,
    SUBJECT,
    NOW_DATE,
    SEND_DATE,
    CALLBACK,
    DEST_COUNT,
    DEST_INFO,
    SMS_MSG,
    RESERVED7,
    RESERVED8,
    RESERVED9,
    KKO_MSG,
    SEND_STATUS
)
SELECT SDK_SMS_SEQ.NEXTVAL 
    , USER_ID
    , RESERVED1
    , RESERVED2
    , RESERVED5
    , SCHEDULE_TYPE
    , SUBJECT
    , NOW_DATE
    , SEND_DATE
    , CALLBACK
    , DEST_COUNT
    , DEST_INFO
    , SMS_MSG
    , RESERVED7
    , RESERVED8
    , RESERVED9
    , KKO_MSG
    , SEND_STATUS
FROM 
(
-- 단문 알림톡 
SELECT 'kjm0826' AS USER_ID
      , '001' AS RESERVED1
      , '7777' AS RESERVED2
      , A.RESERVED5
      , '0' AS SCHEDULE_TYPE
      , '[크린토피아] 알림톡' AS SUBJECT
      , TO_CHAR(SYSDATE, 'YYYYMMDDHH24MISS') AS NOW_DATE
      , TO_CHAR(SYSDATE, 'YYYYMMDDHH24MISS') AS SEND_DATE
      , '15774560' AS CALLBACK
      , 1 AS DEST_COUNT
      , 'X^01033576026' AS DEST_INFO
      , A.SMS_MSG  AS SMS_MSG
      , SUBSTR(B.KKO_BTN_URL, 0, INSTR(B.KKO_BTN_URL, '/', -1)) AS RESERVED7
      , SUBSTR(B.KKO_BTN_URL, INSTR(B.KKO_BTN_URL, '/', -1)+1, LENGTH(B.KKO_BTN_URL)) AS RESERVED8
      , B.KKO_BTN_NAME AS RESERVED9
      , B.SEND_MESSAGE AS KKO_MSG
      , '9' AS SEND_STATUS
	    , ROW_NUMBER() OVER(PARTITION BY B.TEMPLATE_CODE ORDER BY B.MESSAGE_SEQNO DESC ) AS RNK
  FROM SMSUSER.SDK_SMS_REPORT_2206_OLD A 
  INNER JOIN SMSUSER.TSMS_AGENT_MESSAGE_LOG_202206 B ON A.MSG_ID = B.MSG_ID
  WHERE B.SEND_RESULT_CODE1 = 'OK'
)
WHERE RNK = 1;
```

<br />

>  [AS-IS] 장문 알림톡 [TO-BE] 환경 발송 테스트 

```sql

INSERT INTO SMSUSER.SDK_SMS_SEND
(
    MSG_ID,
    USER_ID,
    RESERVED1,
    RESERVED2,
    RESERVED5,
    SCHEDULE_TYPE,
    SUBJECT,
    NOW_DATE,
    SEND_DATE,
    CALLBACK,
    DEST_COUNT,
    DEST_INFO,
    SMS_MSG,
    RESERVED7,
    RESERVED8,
    RESERVED9,
    KKO_MSG,
    SEND_STATUS,
    CDR_ID
)
SELECT SDK_MMS_SEQ.NEXTVAL 
    , USER_ID
    , RESERVED1
    , RESERVED2
    , RESERVED5
    , SCHEDULE_TYPE
    , SUBJECT
    , NOW_DATE
    , SEND_DATE
    , CALLBACK
    , DEST_COUNT
    , DEST_INFO
    , SMS_MSG
    , RESERVED7
    , RESERVED8
    , RESERVED9
    , KKO_MSG
    , SEND_STATUS
    , 'MS02'
FROM 
(
-- 장문 알림톡 
SELECT 'kjm0826' AS USER_ID
      , '001' AS RESERVED1
      , '7777' AS RESERVED2
      , A.RESERVED5
      , '0' AS SCHEDULE_TYPE
      , '[크린토피아] 알림톡' AS SUBJECT
      , TO_CHAR(SYSDATE, 'YYYYMMDDHH24MISS') AS NOW_DATE
      , TO_CHAR(SYSDATE, 'YYYYMMDDHH24MISS') AS SEND_DATE
      , '15774560' AS CALLBACK
      , 1 AS DEST_COUNT
      , 'X^01033576026' AS DEST_INFO
      , A.MMS_MSG  AS SMS_MSG
      , SUBSTR(B.KKO_BTN_URL, 0, INSTR(B.KKO_BTN_URL, '/', -1)) AS RESERVED7
      , SUBSTR(B.KKO_BTN_URL, INSTR(B.KKO_BTN_URL, '/', -1)+1, LENGTH(B.KKO_BTN_URL)) AS RESERVED8
      , B.KKO_BTN_NAME AS RESERVED9
      , B.SEND_MESSAGE AS KKO_MSG
      , '9' AS SEND_STATUS
	    , ROW_NUMBER() OVER(PARTITION BY B.TEMPLATE_CODE ORDER BY B.MESSAGE_SEQNO DESC ) AS RNK
  FROM SMSUSER.SDK_MMS_REPORT_2206_OLD A 
  INNER JOIN SMSUSER.TSMS_AGENT_MESSAGE_LOG_202206 B ON A.MSG_ID = B.MSG_ID
  WHERE B.SEND_RESULT_CODE1 = 'OK'
)
WHERE RNK = 1;
```
