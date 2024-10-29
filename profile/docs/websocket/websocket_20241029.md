# 웹소켓 규격 정리

## Desc
- tetra version: 0.3.4
- socket version: socket.io 4.7.2
- socket address: 'http://{로봇주소}'

```js
/*
 * javascript 에서 사용방법 예시 
 */

import io from 'socket.io-client';

const socketHost = process.env.API_HOST || '192.168.50.2';
const socket = io(socketHost);

socket.on("disconnect", ()=>{
    // 소켓 연결이 끊어졌을 경우
});

socket.on("connect", ()=>{
    // 소켓 연결이 됐을 경우
});

socket.on('reconnect', ()=>{
    // 소켓이 다시 연결 됐을 경우
});

// 소켓 이벤트 구독
socket.on("{구독할 소켓명}", data=>{
    if(data) console.log('hello socket.');
});

// 소켓 이벤트 발행
socket.emit("{발행할 소켓명}", false);

// 소켓 연결 해제
socket.disconnect();
```

```js
/*
 * javascript 에서 map 데이터를 불러와 이미지를 만드는 예제
 */

import io from 'socket.io-client';

const socketHost = '192.168.50.2'; // 로봇 ip
const socket = io(socketHost); // socket 생성

// 맵 생성 로직
socket.on("robot:map", (socketData)=> {
    const width = socketData.info.width;
    const height =  socketData.info.height;
    const mapData =  socketData.data;
    const x = - socketData.info.origin.position.x;
    const y = - socketData.info.origin.position.y;

    const canvas = document.createElement('canvas'); // 캔버스 생성
    const ctx = canvas.getContext('2d');

    canvas.width = width;
    canvas.height = height;

    // 이미지 생성 로직
    const img = ctx.getImageData(0, 0, width, height);
    for (let row = 0; row < (height-1); row++) {
        for (let col = 0; col < (width-1); col++) {
            const mapI = col + ((height - row) * width);
            const pos = ((row * width) + col) * 4;
            let data = mapData[mapI];
            if (data === 100) {
            data = 0;
            } else if (data === 0) {
            data = 255;
            } else if (data >= 67 && data <= 99) {
            data = 50;
            } else if (data >= 51 && data <= 66) {
            data = 100;
            } else if (data >= 34 && data <= 50) {
            data = 150;
            } else if (data >= 1 && data <= 33) {
            data = 200;
            } else {
            data = 127;
            }
            img.data[pos] = data;
            img.data[pos + 1] = data;
            img.data[pos + 2] = data;
            img.data[pos + 3] = maxValue;
        }
    }
    ctx.putImageData(img, 0, 0); // 캔버스에 이미지 적용
});
```

## 구독 가능한 이벤트 리스트 
1. robot:time (data: '{robot_system_time}')
   - 로봇에 설정 되있는 시간을 주기적으로 알림
   
2. autoDocking (data: '{location_id}/{location_name}')
    - 배터리 부족 시 충전 알림 (위치 아이디와 이름를 데이터로 보냄) 

3. job:update
    - job 이 새로 업데이트 됐음을 알림

4. job:working
    - job 실행중 사용자에 의해 goto 명령이 내려질 경우 불가함을 알림 

5. job:goto_location_id (data: '{current_location_id}/{current_work_id}/{current_job_id}')
    - 태스크 중 가야하는 위치 알림 (위치 id, work id(플랜), job id(태스크)를 데이터로 보냄)

6. job:work_finish (data: '{job_done_count}/{job_repeat_count}')
    - 태스크 중 하나의 플랜이 끝났음을 알림

7. robot:battery (data: {'battery': '{current_battery}'})
    - 로봇의 배터리 잔량을 알림

8. robot:load (data: {'load': '{load_yn}'})
    - 로봇의 적제 옵셥이 있을 시 적재 유무를 알림

9. robot:emg (data: {'emg': '{emg_yn}'})
    - 로봇의 EMG 버튼 눌림을 알림

10. robot:speed (data: {'speed': '{robot_linear_speed}'})
    - 로봇의 선 속도를 알림 

11. robot:bumper (data: {'bumper': '{bumper_yn}'})
    - 로봇의 bumper 눌림을 알림

12. robot:gpio (data: {'gpio': {'Input0': '{input0_yn}', ... 'Output7': 'output7_yn'}})
    - 로봇의 GPIO 신호 상태를 알림

13. robot:sys_power (data: {'sys_power': '{system_power_current}'})
    - 로봇의 현재 사용되는 전류량을 알림

14. robot:localPlan (data: {'localPlan': [{'position': {'x': '{x}', 'y': '{y}', 'z': '{z}'}, 'orientation': {'x': '{x}', 'y': '{y}', 'z': '{z}', 'w': '{w}'}}, ...]})
    - 로봇의 로컬 경로를 알림

15. robot:globalPlan (data: {'globalPlan': [{'position': {'x': '{x}', 'y': '{y}', 'z': '{z}'}, 'orientation': {'x': '{x}', 'y': '{y}', 'z': '{z}', 'w': '{w}'}}, ...]})
    - 로봇의 글로벌 경로를 알림

16. docking:available (data: '[{'id': '{marker_id}'}, ...]')
    - 도킹 가능한 마커의 리스트 알림

17. robot:result (data: {'result': '{move_base_result}'})
    - 로봇 이동 명령시 로봇의 이동 상태 알림

18. robot:tf (data: {'translation': {'x': '{x}', 'y': '{y}', 'z': '{z}'}, 'rotation': {'x': '{x}', 'y': '{y}', 'z': '{z}', 'w': '{w}'}})
    - 로봇의 현재 좌표 알림

19. docking
    - 로봇이 됐을 때 알림

20. robot:dockingStatus (data: {'status': '{docking_status}'})
    - 로봇과 스테이션의 통신 상태를 알림

21. robot:map (data: {'header': '{header}', 'info': '{info}', 'data': ['{data}', ...]})
    - 로봇의 맵 데이터 (자세한 내용은 [https://docs.ros.org/en/melodic/api/nav_msgs/html/msg/OccupancyGrid.html](https://docs.ros.org/en/melodic/api/nav_msgs/html/msg/OccupancyGrid.html) 참조)

22. robot:enterArea (data: {'type': 'slow', 'speed': '{area_speed}'})
    - 로봇이 감속구간에 들어 왔음을 알림

23. robot:exitArea (data: {'type': 'slow', 'speed': '{default_speed}'})
    - 로봇이 감속구간에서 벗어났음을 알림

24. robot:roll (data: {'roll': '{roll_float_value}'})
    - 로봇의 roll 각도 알림

25. robot:pitch (data: {'pitch': '{pitch_float_value}'})
    - 로봇의 pitch 각도 알림

26. robot:yaw (data: {'yaw': '{yaw_float_value}'})
    - 로봇의 yaw 각도 알림

## 발행 가능한 이벤트 리스트 

1. robot:cmd_velocity (data: {'linear': {'x': '{x}', 'y': '{y}', 'z': '{z}'}, 'angular': {'x': '{x}', 'y': '{y}', 'z': '{z}'}})
    - 로봇의 선속도와 각속도를 발행 (발행시 지정된 속도로 움직임)

