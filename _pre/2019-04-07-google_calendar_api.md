---
title:  "구글캘린더API로 야구경기 일정 가져오기"
subtitle: "Google_Calendar_API"
search: false
categories:
  - geultto
tags: 글또
comments: true
---

본 글은 구글 캘린더API를 연동하는 방법에 대해서 설명하고, 추가로 해당 캘린더의 장소를 함께 데이터로 추출하는 과정을 진행한다.

*캘린더 정보는 린더(https://linder.kr) 에서 가져왔다.*

본 링크를 토대로 진행을 했다(Python)
: https://developers.google.com/calendar/quickstart/python

1. Google Calendar API 활성화
해당 페이지에서 crendential.json을 발급 받으면 된다.
(또는 https://console.developers.google.com/?hl=ko 에서도 가능)

2. 구글 클라이언트 라이브러리 설치

`pip install --upgrade google-api-python-client google-auth-httplib2 google-auth-oauthlib`

3. 샘플 실행
아래 credentials.json 만 정상적으로 잘 넣어주면, 정상적으로 실행가능하다.

```python
from __future__ import print_function
import datetime
import pickle
import os.path
from googleapiclient.discovery import build
from google_auth_oauthlib.flow import InstalledAppFlow
from google.auth.transport.requests import Request

# If modifying these scopes, delete the file token.pickle.
SCOPES = ['https://www.googleapis.com/auth/calendar.readonly']

def main():
    """Shows basic usage of the Google Calendar API.
    Prints the start and name of the next 10 events on the user's calendar.
    """
    creds = None
    # The file token.pickle stores the user's access and refresh tokens, and is
    # created automatically when the authorization flow completes for the first
    # time.
    if os.path.exists('token.pickle'):
        with open('token.pickle', 'rb') as token:
            creds = pickle.load(token)
    # If there are no (valid) credentials available, let the user log in.
    if not creds or not creds.valid:
        if creds and creds.expired and creds.refresh_token:
            creds.refresh(Request())
        else:
            flow = InstalledAppFlow.from_client_secrets_file(
                'credentials.json', SCOPES)
            creds = flow.run_local_server()
        # Save the credentials for the next run
        with open('token.pickle', 'wb') as token:
            pickle.dump(creds, token)

    service = build('calendar', 'v3', credentials=creds)

    # Call the Calendar API
    now = datetime.datetime.utcnow().isoformat() + 'Z' # 'Z' indicates UTC time
    print('Getting the upcoming 10 events')
    events_result = service.events().list(calendarId='primary', timeMin=now,
                                        maxResults=10, singleEvents=True,
                                        orderBy='startTime').execute()
    events = events_result.get('items', [])

    if not events:
        print('No upcoming events found.')
    for event in events:
        start = event['start'].get('dateTime', event['start'].get('date'))
        print(start, event['summary'])

if __name__ == '__main__':
    main()
```

4. 샘플 실행.

  `$ python quickstart.py`


### 캘린더 정보 가져오기
캘린더를 린더를 통해 내 구글캘린더에 등록을 시킨 후, 등록된 캘린더 정보를 가져오는 방식을 취했다.

- 필요한 정보는 `#이벤트타입/이벤트명/이벤트시작datetime/이벤트종료datetime/주소/위경도값`
이고, 해당 정보를 통해 사람이 많이 모이는 이벤트 정보를 확보해서 수요 등을 미리 파악할 수 있는 용도를 위해서다.

- 기본 베이스는 샘플코드를 기반으로 했기 때문에, 주요 캘린더함수를 설명한다.
(자세한 캘린더 함수는 아래 링크에서 확인 할 수 있다. : https://developers.google.com/calendar/v3/reference/events/list)

### events() 내장 함수
    ['delete', -- 캘린더 삭제
    'get', -- 캘린더 가져오기
    'import_', -- 캘린더 외부정보 삽입
    'insert', -- 캘린더 삽입
    'instances',
    'instances_next',
    *'list',* -- 캘린더 리스트 정보 (본 포스팅에서 사용하는 함수)
    'list_next',
    'move', -- 캘린더 이동
    'patch',
    'quickAdd',
    'update', -- 캘린더 업데이트(수정)
    'watch'] -- 캘린더 권한

- list() 파라미터

      1. 캘린더ID 'primary'(default)
      2. 기준시간 timeMin=now / timeMax
      3. 출력갯수 maxResults=10,
      4. 단일이벤트여부 singleEvents=True(default=False) ,
      5. 나열순서 orderBy='startTime

와 같으며, 1번을 제외하면 나머지는 '선택'파라미터값이다.

- 1번을 통해 특정캘린더 값을 가져올 수 있으며
- 2번을 통해 현재 이전/이후 캘린더 값의 바운더리를 정할 수 있다
- 3번은 API호출의 수를 정하는 값이다.
- 4번은 이벤트가 1회성인지를 표시하는 것이다.
- 5번은 이벤트가 여러개 있을때 순서를 표현.

캘린더ID는 해당 스크린샷에서 가져올 수 있다.
![cal](https://www.dropbox.com/s/g9uvav4rpisl3ak/cal_1.png?dl=1)

```python
events_result = service.events().list(calendarId='primary', timeMin=now,
                                    maxResults=10, singleEvents=True,
                                    orderBy='startTime').execute()
events = events_result.get('items', [])
```

본 글에선 두산베어스의 경기 일정을 가져와보았다.
생각보다 간단하게 사용가능한 구글 캘린더라서, 여기다 구글맵스까지 활용해서, 위경도정보까지 불러와보았다.(구글맵스에 대한 설명은 생략했다.)

![cal](https://www.dropbox.com/s/4mxsuxwfvmmi1ef/cal_2.png?dl=1)

실제 날짜와 비교했을때 잘 가져와지는 것을 확인했다.
![cal](https://www.dropbox.com/s/1ydg5774bundjy9/cal_3.png?dl=1)

해당 소스코드는 파일로 공유한다. [실습파일링크](https://www.dropbox.com/s/zqvbs0h7nf60406/Google_Calendar%20API%20%EC%97%B0%EB%8F%99.ipynb?dl=1)

캘린더 활용안은 다양하게 사용할 수 있을 것 같다. ~~이제 이걸로 아이돌 공연정보도 가져오는걸로...~~
