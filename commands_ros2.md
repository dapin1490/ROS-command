<h1>ROS2 커맨드</h1>

- [팁](#팁)
- [007 패키지 설치와 노드 실행](#007-패키지-설치와-노드-실행)
- [008 노드와 데이터 통신](#008-노드와-데이터-통신)
- [009 토픽](#009-토픽)
- [010 서비스](#010-서비스)
- [011 액션](#011-액션)
- [013 파라미터](#013-파라미터)
- [016 인터페이스](#016-인터페이스)
- [020 파일 시스템](#020-파일-시스템)
- [021 빌드 시스템과 빌드 툴](#021-빌드-시스템과-빌드-툴)
- [024+025 ROS 프로그래밍 기초](#024025-ros-프로그래밍-기초)
  - [py `package.xml` 커스텀](#py-packagexml-커스텀)
  - [cpp `package.xml` 커스텀](#cpp-packagexml-커스텀)
  - [py `setup.py` 커스텀](#py-setuppy-커스텀)
  - [cpp `CMakeLists.txt` 커스텀](#cpp-cmakeliststxt-커스텀)
- [027 토픽, 서비스, 액션 인터페이스](#027-토픽-서비스-액션-인터페이스)
  - [`package.xml` 커스텀](#packagexml-커스텀)
  - [`CMakeLists.txt` 커스텀](#cmakeliststxt-커스텀)
- [028+034 ROS 2 패키지 설계](#028034-ros-2-패키지-설계)
  - [topic\_service\_action\_rclcpp\_example 패키지 구성](#topic_service_action_rclcpp_example-패키지-구성)
  - [topic\_service\_action\_rclcpp\_example 패키지 실행](#topic_service_action_rclcpp_example-패키지-실행)

---

기본 참고 링크: [000 로봇 운영체제 ROS 강좌 목차](https://cafe.naver.com/openrt/24070)

예제 프로젝트 폴더 경로: `~/robot_ws/`


## 팁
* 파일을 저장했는데 어디 있는지 모르겠다: `ls *.sh` 또는 `ls -ltr *.sh` 사용하면 홈디렉토리 내에 있는 모든 배쉬 파일 목록 조회 가능
* 어떤 폴더 안에 저장했는데 못찾겠다: `ls -ltr */*.sh` 사용하면 한 단계 들어간 폴더 내의 모든 파일 조회
* 잘못 설치한 폴더 삭제하기: `sudo rm -r [지울 폴더명]`
* 패키지 실행자 목록 보기: `ros2 pkg executables <pkg_name>`
* 검색 결과 중에서 검색하기: `[검색 명령어] | grep [검색어]`
* 대기 시간 줄 때는: `sleep 1 `
* 폴더 내 모든 압축파일을 압축파일 폴더명으로 압축 해제하기

```shell
#!/bin/bash

for f in *.zip
do
    name="${f:0:8}"
    echo $name
    7z e "$f" -o"$name"
done

rm *.zip
```

## 007 패키지 설치와 노드 실행
* 설치된 패키지 확인 및 검색: `ros2 pkg list | grep [검색어]`
* 사용 가능한 실행자(실행 가능한 노드) 검색: `ros2 pkg executables turtlesim`

<!--  -->

* 거북이 소환: `ros2 run turtlesim turtlesim_node`
* 컨트롤러 소환: `ros2 run turtlesim turtle_teleop_key`

`ros2 [조회 대상] list`
* `node`: 사용중인 노드 목록
* `topic`: 사용중인 토픽 목록
* `service`: 사용중인 서비스
* `action`: 사용중인 액션

<!--  -->

* `rqt_graph`: 노드와 토픽의 그래프 뷰

## 008 노드와 데이터 통신
* 새 창에 새 거북이 소환, 이름은 "new_turtle": `ros2 run turtlesim turtlesim_node __node:=new_turtle`
* 특정 노드(이름으로) 정보 보기: `ros2 node info /turtlesim`  
    -> `new_turtle`이라는 이름으로 노드를 만들었을 경우 `/new_turtle`로 검색해야 함.

## 009 토픽
* 토픽 목록: `ros2 topic list -t`
* 정보: `ros2 topic info /turtle1/cmd_vel`
* 내용: `ros2 topic echo /turtle1/cmd_vel`
* 대역폭: `ros2 topic bw /turtle1/cmd_vel`
* 주기: `ros2 topic hz /turtle1/cmd_vel`
* 지연 시간: `ros2 topic delay /TOPIC_NAME`
* 토픽 목록:  
            `/parameter_events [rcl_interfaces/msg/ParameterEvent]`  
            `/rosout [rcl_interfaces/msg/Log]`  
            `/turtle1/cmd_vel [geometry_msgs/msg/Twist]`  
            `/turtle1/color_sensor [turtlesim/msg/Color]`  
            `/turtle1/pose [turtlesim/msg/Pose]`  
            pose와 color_sensor는 구독 상태가 아니라고 뜸. 입력해도 실행중인 노드에 전달되지 않음. 왜 이러는지는 모르겠으나 방향 바꾸려면 cmd_vel 권장

<!--  -->

토픽 발행
* 템플릿: `ros2 topic pub <topic_name> <msg_type> "<args>"`
* 사분원: `ros2 topic pub --once /turtle1/cmd_vel geometry_msgs/msg/Twist "{linear: {x: 2.0, y: 0.0, z: 0.0}, angular: {x: 0.0, y: 0.0, z: 3.14}}"`  
    180도 회전: `z: 3.14`  
    360도 회전: `z: 6.28`  
    -> ros2에서 각도는 모두 180도 = 3.14라고 두고 계산하면 된다.
* 무한반복 원: `ros2 topic pub --rate 1 /turtle1/cmd_vel geometry_msgs/msg/Twist "{linear: {x: 2.0, y: 0.0, z: 0.0}, angular: {x: 0.0, y: 0.0, z: 3.14}}"`

bag 기록
* 템플릿: `ros2 bag record <topic_name1> <topic_name2> <topic_name3>`  
    한 번에 여러 개의 토픽을 기록할 수 있음. 다만 토픽 외의 것들 (action)은 기록 못 함
* 기본 기록: `ros2 bag record /turtle1/cmd_vel`  
    10초 정도 가만히 냅뒀다가 `Ctrl+C`로 종료
* 기록 정보 조회: `ros2 bag info rosbag2_[기록날짜]/`  
    파일명은 `rosbag2_`까지만 입력하고 탭을 3번 치면 파일명 목록이 나온다. 따라 쓰면 됨. 하나만 있다면 탭 한 번에 바로 완성됨.
* 기록 재생: `ros2 bag play rosbag2_[기록날짜]/`

토픽 해석
* 원본 명령어 : `ros2 topic pub --once /turtle1/cmd_vel geometry_msgs/msg/Twist "{linear: {x: 2.0, y: 0.0, z: 0.0}, angular: {x: 0.0, y: 0.0, z: 3.14}}"`
* `ros2` : ROS2의 명령어를 쓰겠다
* `topic` : 토픽을 쓰겠다
* `pub` : 토픽을 '발행'할 것이다
* `--once` : 한 번만 실행해라
* `/turtle1/cmd_vel` : 토픽을 받을 대상. 거북이를 새로 생성했을 경우 \[turtle1]자리에 새 거북이 이름 써야 함
* `geometry_msgs/msg/Twist` : 토픽 내용
* `"{linear: {x: 2.0, y: 0.0, z: 0.0}, angular: {x: 0.0, y: 0.0, z: 3.14}}"` : 얼마나 진행하고, 얼마나 회전할 것인지
    * `linear x`에 거북이의 이동 거리를 정하고
    * `angular z`에 거북이의 회전 각도를 정함. 라디안 단위.

## 010 서비스
* 서비스 목록 확인: `ros2 service list`
* 서비스 형태 확인: `ros2 service type /[서비스명]`
* 서비스 목록과 형태 확인: `ros2 service list -t`
* 서비스 찾기: `ros2 service find [서비스 형태(예: std_srvs/srv/Empty)]`
* 서비스 요청 템플릿: `ros2 service call <service_name> <service_type> "<arguments>"`
* 서비스 목록:  
            `/clear: std_srvs/srv/Empty`  
            `/kill: turtlesim/srv/Kill`  
            `/reset: std_srvs/srv/Empty`  
            `/spawn: turtlesim/srv/Spawn`  
            `/turtle1/set_pen: turtlesim/srv/SetPen`  
            `/turtle1/teleport_absolute: turtlesim/srv/TeleportAbsolute`  
            `/turtle1/teleport_relative: turtlesim/srv/TeleportRelative`  
            `/turtlesim/describe_parameters: rcl_interfaces/srv/DescribeParameters`  
            `/turtlesim/get_parameter_types: rcl_interfaces/srv/GetParameterTypes`  
            `/turtlesim/get_parameters: rcl_interfaces/srv/GetParameters`  
            `/turtlesim/list_parameters: rcl_interfaces/srv/ListParameters`  
            `/turtlesim/set_parameters: rcl_interfaces/srv/SetParameters`  
            `/turtlesim/set_parameters_atomically: rcl_interfaces/srv/SetParametersAtomically`

<!--  -->

* 서비스 예시  
    * 거북이 펜 지우기: `ros2 service call /clear std_srvs/srv/Empty`
    * 거북이 지우기: `ros2 service call /kill turtlesim/srv/Kill "name: 'turtle1'"`
    * 다 리셋: `ros2 service call /reset std_srvs/srv/Empty`
    * 거북이 펜 꾸미기: `ros2 service call /turtle1/set_pen turtlesim/srv/SetPen "{r: 255, g: 255, b: 255, width: 10}"`

<!--  -->

* 닌자거북이  
    * 기본 거북이 지우기: `ros2 service call /kill turtlesim/srv/Kill "name: 'turtle1'"`
    * 레오나르도: `ros2 service call /spawn turtlesim/srv/Spawn "{x: 5.5, y: 9, theta: 1.57, name: 'leonardo'}"`
    * 라파엘로: `ros2 service call /spawn turtlesim/srv/Spawn "{x: 5.5, y: 7, theta: 1.57, name: 'raffaello'}"`
    * 미켈란젤로: `ros2 service call /spawn turtlesim/srv/Spawn "{x: 5.5, y: 5, theta: 1.57, name: 'michelangelo'}"`
    * 도나텔로: `ros2 service call /spawn turtlesim/srv/Spawn "{x: 5.5, y: 3, theta: 1.57, name: 'donatello'}"`

## 011 액션
* 액션 목록: `ros2 action list -t`
* 액션 정보: `ros2 action info /turtle1/rotate_absolute`
* 액션 목표 전달 템플릿: `ros2 action send_goal <action_name> <action_type> "<values>"`
* 12시 방향으로 회전: `ros2 action send_goal /turtle1/rotate_absolute turtlesim/action/RotateAbsolute "{theta: 1.5708}"`  
    뒤에 `--feedback`을 붙이면 회전하면서 남은 회전량 출력  
    오른쪽 방향(노드의 초기 방향)을 기준으로 180도 회전은 3.14로 놓고 계산하면 됨.

teleop 거북이 회전 각

| | | |
|-|-|-|
| `E`: 2.3562  | `R`: 1.5708     | `T`: 0.7854 |
| `D`: 3.1416  | `F`: force stop | `G`: 0.0 |
| `C`: -2.3562 | `V`: -1.5708    | `B`: -0.7854 |

## 013 파라미터
* 파라미터 리스트: `ros2 param list`
* 특정 파라미터 자세히 보기: `ros2 param list describe /turtlesim [파라미터명]`
* 특정 파라미터 값 얻기(get): `ros2 param get /turtlesim [파라미터명]`
* 특정 파라미터 값 바꾸기(set): `ros2 param set /turtlesim [파라미터명] [값]`
* 바꾼 파라미터 저장하기(결과로 출력되는 파일에 파라미터 상태가 저장됨): `ros2 param dump /turtlesim`

<!--  -->

* 저장된 파라미터 확인하기: `gedit turtlesim.yaml`
* 저장한 파라미터 이용하여 터틀심 생성: `ros2 run turtlesim turtlesim_node --ros-args --params-file ./turtlesim.yaml`
* 파라미터 삭제: `ros2 param delete /turtlesim background_b`  
    삭제한 값은 0이 되는 것이 아니라 초기값으로 설정된 것처럼 취급됨

<!--  -->

* 무슨 명령어를 쳐야 할지 모르겠다: 명령어 뒤에 `-h`를 붙이면 그 다음에 뭘 쓸 수 있는지 보여줌

<!--  -->

질문: 삭제한 파라미터를 다시 추가할 수도 있나?  
답변: 삭제한 시점에 바로 직접 추가하는 것은 안 되고, 터미널 창을 완전히 끈 다음 다시 실행하면 복구된다. 파라미터가 너무 많거나 특정 파라미터가 없을 때 나타나는 변화를 확인하고자 할 때 파라미터를 삭제한다.

## 016 인터페이스
* [016 ROS 2 인터페이스 (interface)](https://cafe.naver.com/openrt/24241)
* 메시지 인터페이스 정보 보기: `ros2 interface show geometry_msgs/msg/Vector3`

`ros2 interface` 에는 show 이외에도 list, package, packages, proto가 있는데  
`list`는 현재 개발 환경의 모든 msg, srv, action 메시지를 보여주며,  
`packages`는 msg, srv, action 인터페이스를 담고 있는 패키지의 목록을 보여준다.  
`package 옵션에 패키지명을 입력하면` 지정한 패키지에 포함된 인터페이스들을 보여주고  
`proto에 특정 인터페이스 형태를 입력하면` 그 인터페이스의 기본 형태를 표시해준다.

* 모든 인터페이스 목록(패키지 + 인터페이스 이름): `ros2 interface list`
* 인터페이스가 있는 패키지 목록: `ros2 interface packages`
* 특정 패키지의 인터페이스 목록: `ros2 interface package turtlesim`
* 인터페이스의 기본 형태 확인: `ros2 interface proto geometry_msgs/msg/Twist`
* 서비스 인터페이스(파일) 목록 보기: `ros2 interface show turtlesim/srv/Spawn.srv`
* 액션 인터페이스(파일) 목록 보기: `ros2 interface show turtlesim/action/RotateAbsolute.action`

## 020 파일 시스템
* 바이너리 설치: `sudo apt install ros-foxy-teleop-twist-joy`
* 소스 코드 설치:  
    ```shell
    cd ~/robot_ws/src
    git clone https://github.com/ros2/teleop_twist_joy.git
    cd ~/robot_ws/
    colcon build --symlink-install --packages-select teleop_twist_joy
    ```

## 021 빌드 시스템과 빌드 툴
* 패키지 생성 템플릿: `ros2 pkg create [패키지이름] --build-type [빌드 타입] --dependencies [의존하는패키지1] [의존하는패키지n]`
* cpp와 py 패키지 생성(`robot_ws/`):  
    `ros2 pkg create test_pkg_rclcpp --build-type ament_cmake`  
    `ros2 pkg create test_pkg_rclpy --build-type ament_python`
* 개인 패키지 생성(`robot_ws/src/`, `my_first_ros_rclcpp_pkg`):  
    `ros2 pkg create [패키지명] --build-type ament_cmake --dependencies rclcpp std_msgs`  
    `ros2 pkg create [패키지명] --build-type ament_python --dependencies rclpy std_msgs`
* 빌드(`robot_ws/`): `colcon build --symlink-install` OR `colcon build --symlink-install --packages-select [패키지 이름]`

## 024+025 ROS 프로그래밍 기초
* 패키지 생성  
    * py: `ros2 pkg create my_first_ros_rclpy_pkg --build-type ament_python --dependencies rclpy std_msgs`
    * cpp: `ros2 pkg create my_first_ros_rclcpp_pkg --build-type ament_cmake --dependencies rclcpp std_msgs`
* 빌드  
    * py: `cd ~/robot_ws && colcon build --symlink-install --packages-select my_first_ros_rclpy_pkg`
    * cpp: `cd ~/robot_ws && colcon build --symlink-install --packages-select my_first_ros_rclcpp_pkg`
* 빌드 템플릿  
    ```shell
    (워크스페이스내의 모든 패키지 빌드하는 방법)
    $ cd ~/robot_ws && colcon build --symlink-install

    (특정 패키지만 빌드하는 방법)
    $ cd ~/robot_ws && colcon build --symlink-install --packages-select [패키지 이름1] [패키지 이름2] [패키지 이름N]

    (특정 패키지 및 의존성 패키지를 함께 빌드하는 방법)
    $ cd ~/robot_ws && colcon build --symlink-install --packages-up-to [패키지 이름]
    ```

* 첫 빌드 후 환경설정 적용  
    `cd ~/robot_ws/install && . local_setup.bash`  
    또는 `. ~/robot_ws/install/local_setup.bash`

* 실행  
    `ros2 run my_first_ros_rclpy_pkg helloworld_subscriber`  
    `ros2 run my_first_ros_rclpy_pkg helloworld_publisher`

### py `package.xml` 커스텀
아래는 템플릿일 뿐이니 실제 사용하는 패키지와 의존성에 따라 바꿔 넣어야 함

```xml
<depend>rclpy</depend>
<depend>std_msgs</depend>
```

### cpp `package.xml` 커스텀
아래는 템플릿일 뿐이니 실제 사용하는 패키지와 의존성에 따라 바꿔 넣어야 함

```xml
<buildtool_depend>rosidl_default_generators</buildtool_depend>
<exec_depend>builtin_interfaces</exec_depend>
<exec_depend>rosidl_default_runtime</exec_depend>
<member_of_group>rosidl_interface_packages</member_of_group>

<depend>rclcpp</depend>
<depend>std_msgs</depend>
<depend>msg_srv_action_interface_example</depend>
```

### py `setup.py` 커스텀
아래는 템플릿일 뿐이니 실제 사용하는 패키지와 의존성에 따라 바꿔 넣어야 함

```py
package_name = 'my_first_ros_rclpy_pkg'
setup(
    # 앞부분 생략
    entry_points={
        'console_scripts': [
            'helloworld_publisher = my_first_ros_rclpy_pkg.helloworld_publisher:main',
            'helloworld_subscriber = my_first_ros_rclpy_pkg.helloworld_subscriber:main',
        ],
    },
)
```

### cpp `CMakeLists.txt` 커스텀
아래는 템플릿일 뿐이니 실제 사용하는 패키지와 의존성에 따라 바꿔 넣어야 함

```cpp
# Find dependencies
find_package(ament_cmake REQUIRED)
find_package(rclcpp REQUIRED)
find_package(std_msgs REQUIRED)
find_package(msg_srv_action_interface_example REQUIRED)

# Build
add_executable(helloworld_publisher src/helloworld_publisher.cpp)
ament_target_dependencies(helloworld_publisher rclcpp std_msgs)

add_executable(helloworld_subscriber src/helloworld_subscriber.cpp)
ament_target_dependencies(helloworld_subscriber rclcpp std_msgs)

# Install
install(TARGETS
  helloworld_publisher
  helloworld_subscriber
  DESTINATION lib/${PROJECT_NAME})
```

## 027 토픽, 서비스, 액션 인터페이스
* 인터페이스 패키지 만들기 **(C++ 패키지라는 점에 주의)**  
    ```shell
    cd ~/robot_ws/src
    ros2 pkg create --build-type ament_cmake msg_srv_action_interface_example
    cd msg_srv_action_interface_example
    mkdir msg srv action
    ```  
    이후 참고 링크에서 요구하는 파일을 각 폴더에 생성, 요구하는 내용으로 파일 저장
* 빌드: `cw && cbp msg_srv_action_interface_example`
* 각종 오류로 진행이 불가할 때 마지막 방법:  
    ```shell
    cw && cd src && git clone https://github.com/robotpilot/ros2-seminar-examples
    cw && cd src && rm -r msg_srv_action_interface_example
    cw && cd src && mv ros2-seminar-examples/msg_srv_action_interface_example/ .
    ```  
    이후 빌드

### `package.xml` 커스텀
아래는 템플릿일 뿐이니 실제 사용하는 패키지와 의존성에 따라 바꿔 넣어야 함

```xml
<buildtool_depend>rosidl_default_generators</buildtool_depend>
<exec_depend>builtin_interfaces</exec_depend>
<exec_depend>rosidl_default_runtime</exec_depend>
<member_of_group>rosidl_interface_packages</member_of_group>
```

### `CMakeLists.txt` 커스텀
아래는 템플릿일 뿐이니 실제 사용하는 패키지와 의존성에 따라 바꿔 넣어야 함

```cpp
set(msg_files
  "msg/ArithmeticArgument.msg"
)

set(srv_files
  "srv/ArithmeticOperator.srv"
)

set(action_files
  "action/ArithmeticChecker.action"
)

rosidl_generate_interfaces(${PROJECT_NAME}
  ${msg_files}
  ${srv_files}
  ${action_files}
  DEPENDENCIES builtin_interfaces
)

ament_export_dependencies(rosidl_default_runtime)
```

## 028+034 ROS 2 패키지 설계
* 소스 코드 다운로드 및 빌드  
    ```shell
    cw && cd src
    git clone https://github.com/robotpilot/ros2-seminar-examples.git
    cw && colcon build --symlink-install
    echo 'source ~/robot_ws/install/local_setup.bash' >> ~/.bashrc
    source ~/.bashrc
    ```

### topic_service_action_rclcpp_example 패키지 구성
1. 토픽의 관점에서, argument는 퍼블리셔(보내는 쪽), calculator는 서브스크라이버(받는 쪽). 퍼블리셔는 일방적으로 메시지를 전송하며, 서브스크라이버는 메시지 수신 여부와 관계 없이 별도 응답을 보내지 않는다.
2. 서비스의 관점에서, operator는 클라이언트(보내는 쪽), calculator는 서버(받는 쪽). 클라이언트는 서버에게 request를 보내고, 서버는 이를 받고 응답해야 한다.
3. 액션의 관점에서, checker는 클라이언트(보내는 쪽), calculator는 서버(받는 쪽). 클라이언트가 먼저 액션의 목표(goal)를 보내고, 서버는 해당 목표를 받아 클라이언트에게 연산 과정(feedback, 중간 결과값)과 연산 결과(result)를 보낸다.

### topic_service_action_rclcpp_example 패키지 실행
* 토픽 서브스크라이버, 서비스 서버, 액션 서버 실행: `ros2 run topic_service_action_rclpy_example calculator`
* 토픽 퍼블리셔 실행: `ros2 run topic_service_action_rclpy_example argument`
* 서비스 클라이언트 실행: `ros2 run topic_service_action_rclpy_example operator`
* 액션 클라이언트 실행: `ros2 run topic_service_action_rclpy_example checker`
* 런치 파일 실행: `ros2 launch topic_service_action_rclpy_example arithmetic.launch.py`
