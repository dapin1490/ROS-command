<h1>ROS 커맨드</h1>

- [팁](#팁)
- [기본 실행 명령어](#기본-실행-명령어)
- [bringup](#bringup)
- [topic monitor](#topic-monitor)
- [teleop](#teleop)
- [SLAM](#slam)
- [save map](#save-map)
- [Navigation](#navigation)
- [Simulation](#simulation)
  - [SLAM Simulation](#slam-simulation)


기본 참고 링크: [Quick Start Guide](https://emanual.robotis.com/docs/en/platform/turtlebot3/quick-start/)

* 와이파이 이름: supermobilityclass
* 와이파이 비번: pw12345678
* 교수님 내선번호: 7609
* 터틀봇 비밀번호: `turtlebot`

## 팁
[[ROS] ROS 명령어 정리](https://happyobo.github.io/ros/ros6-post/)

* 실행중인 토픽 리스트 확인: `rostopic list`
* 토픽 내용 출력: `rostopic echo <topic_name>`

<!--  -->

* IP 주소 찾기 (IP 주소는 대체로 항상 바뀐다. 바뀌지 않는 주소를 원한다면 MAC 주소를 외워둘 것.)
    * 윈도우 PC와 리눅스 PC가 별도로 있을 경우
        1. 윈도우 PC 설정에서 네트워크 > 모바일 핫스팟 설정을 찾아 와이파이 이름과 비밀번호, 주파수 대역을 선택하고 실행한다. 설정 창은 그대로 열어둔다.
        2. 만약 터틀봇이 마지막으로 연결했던 와이파이 정보와 달라졌을 경우 SD카드를 꺼내 리눅스 PC에 연결하고 재설정한다. [Configure the WiFi Network Setting](https://emanual.robotis.com/docs/en/platform/turtlebot3/sbc_setup/#configure-the-wifi-network-setting-1)
        3. 리눅스 PC와 터틀봇 모두 윈도우 PC가 호스팅한 와이파이로 연결하면 윈도우 PC의 모바일 핫스팟 설정 창에 연결된 기기 목록이 뜬다.
        4. 연결된 기기 목록의 IP 주소를 보고 .bashrc 파일을 수정한 후 [bringup](https://github.com/dapin1490/ROS-command/blob/main/commands_ros1.adoc#bringup)을 진행하면 된다.
    * 리눅스 PC만 있을 경우
        1. 리눅스 PC에 nmap을 설치한다: `sudo apt install nmap`. `snap`을 통한 설치도 가능하나 이 경우 `sudo apt update`로 업데이트가 안된다는 점을 고려해야 한다.
        2. 위의 경우와 마찬가지로 와이파이 설정 후 터틀봇과 함께 서로 동일한 와이파이에 연결한다.
        3. 터미널에서 다음 명령어를 실행한다: `nmap -sP 192.168.0.0/24`. 만약 실행이 잘 안된다면 앞에 `sudo`를 붙여본다.
            * `-sP` 뒤에 오는 부분은 리눅스 PC의 IP 주소를 기준으로 한다.
            * 앞의 세 숫자는 그대로 쓴다.
            * 마지막 숫자는 0으로 쓴다.
            * 마지막 숫자의 뒤에 붙는 `/24`는 넷마스크의 비트 수로, IP주소가 192 이상의 숫자로 시작한다면 그대로 쓰면 된다. -> 자세히 알고자 한다면 서브넷마스크와 CIDR에 대해 검색하면 된다.
        4. `nmap`의 실행이 끝나면 현재 리눅스 PC와 동일한 네트워크에 연결된 기기들의 IP 주소가 모두 출력된다. 그중 터틀봇의 것을 찍어서 찾아내면 된다.
        5. 농담이 아니라 대체로 찍는 것밖에 방법이 없다. 게이트웨이, 브로드캐스트, 리눅스 PC의 주소 등 기본적으로 확실하게 터틀봇이 아닌 것을 제외한 나머지 중에서 매번 연결을 시도하며 찾아야 한다.

<!--  -->

* ssh 연결
    * 터틀봇의 현재 IP 주소를 반드시 알고 있어야 한다.
    * 터틀봇의 이름과 비밀번호는 기본값이라는 전제로 작성했으며, 바꿨을 경우 해당 사항을 적용해서 써야 한다.
    * 리눅스 터미널에서 다음 명령어를 실행한다: `ssh ubuntu@192.168.1.n`. `@` 앞에는 터틀봇의 이름을 쓰고, 뒤에는 터틀봇의 IP주소를 쓴다.
    * 대략 서너 줄 정도의 긴 텍스트와 함께 마지막에 `(yes/no/fingerprint)`를 묻는 프롬프트가 나온다면 yes 입력
    * 이후 터틀봇의 비밀번호를 치면 된다.
    * 연결에 성공할 경우 출력문 맨 아랫줄에 마지막 접속 일시와 마지막으로 로그인한 리눅스 PC의 IP 주소가 나온다. 이후 해당 터미널은 터틀봇의 터미널로 사용된다. 리눅스 PC의 터미널을 쓰고자 한다면 새 탭이나 새 창을 열면 된다.

## 기본 실행 명령어
dapin: `$ roscore`  
dapin: `$ ssh ubuntu@{IP_ADDRESS_OF_RASPBERRY_PI}`  
ubuntu: `$ export TURTLEBOT3_MODEL=waffle_pi`

## bringup
ubuntu: `$ roslaunch turtlebot3_bringup turtlebot3_robot.launch`  
pc와 터틀봇의 bashrc에서 ip를 설정해주어야 한다. pc에서는 호스트와 마스터가 둘 다 자신의 ip, 터틀봇은 마스터의 ip가 pc, 호스트의 ip가 터틀봇.  
after edit `~/.bashrc`, you must write `source ~/.bashrc`

## topic monitor
1. PC에서 아래 명령어로 rqt를 실행합니다. 토픽 모니터 창이 표시되지 않으면 `plugin` -> `Topics` -> `Topic Monitor`를 선택합니다.  
    `$ rqt`
2. Topic Monitor가 로드되면 Topic 값이 모니터링되지 않습니다. 토픽을 모니터링하려면 각 토픽 옆의 확인란을 클릭합니다.
3. 더 자세한 주제 메시지를 보려면 확인란 옆의 ▶ 아이콘을 클릭합니다.
4. `/battery_state`는 현재 배터리 전압 및 남은 용량 등 배터리 상태와 관련된 메시지를 표시합니다.
5. `/diagnostics` 은 MPU9250, 다이나믹셀-X, HLS-LFCD-LDS, 배터리, OpenCR 등 TurtleBot3에 연결된 구성 요소의 상태를 메시지로 표시합니다.
6. `/odom`은 TurtleBot3의 주행 거리 측정 메시지를 나타냅니다. 이 항목에는 인코더 데이터에 의한 방향과 위치가 있습니다.
7. `/sensor_state`는 인코더 값, 배터리 및 토크를 메시지로 표시합니다.
8. `/scan`은 각도_최대 및 최소, 범위_최대 및 최소, 범위 표시, 범위와 같은 모든 LDS 데이터를 메시지로 표시합니다.

## teleop
dapin: `$ roslaunch turtlebot3_teleop turtlebot3_teleop_key.launch`

```shell
w/x : increase/decrease linear velocity
a/d : increase/decrease angular velocity
space key, s : force stop
CTRL-C to quit
```

## SLAM
사전 준비: `roscore` -> `ssh` -> `export TURTLEBOT3_MODEL=waffle_pi` -> `roslaunch turtlebot3_bringup turtlebot3_robot.launch`

새 터미널:  
`$ export TURTLEBOT3_MODEL=waffle_pi`  
`$ roslaunch turtlebot3_slam turtlebot3_slam.launch`

새 터미널:  
`$ export TURTLEBOT3_MODEL=waffle_pi`  
`$ roslaunch turtlebot3_teleop turtlebot3_teleop_key.launch`

## save map
`$ rosrun map_server map_saver -f ~/map`  
`-f`: 지정된 파일 경로에 map을 저장함

## Navigation
NOTE: 원격 PC에서 내비게이션을 실행하세요.  
작업을 실행하기 전에 반드시 TurtleBot3에서 불러오기를 실행하세요.  
내비게이션은 SLAM에서 생성한 지도를 사용합니다. 내비게이션을 실행하기 전에 지도를 준비해 주세요.

Navigation
1. Run Navigation Nodes
    1. `$ roscore`
    2. `$ ssh ubuntu@192.168.1.X`  
        `$ export TURTLEBOT3_MODEL=waffle_pi`  
        `$ roslaunch turtlebot3_bringup turtlebot3_robot.launch`
    3. `$ export TURTLEBOT3_MODEL=waffle_pi`  
        `$ roslaunch turtlebot3_navigation turtlebot3_navigation.launch map_file:=$HOME/map.yaml`
2. Estimate Initial Pose
    1. RViz 메뉴에서 `2D Pose Estimate` 버튼을 클릭합니다.
    2. 실제 로봇이 위치한 지도를 클릭하고 큰 녹색 화살표를 로봇이 향하는 방향으로 드래그합니다.
    3. 저장된 지도에 LDS 센서 데이터가 오버레이될 때까지 1단계와 2단계를 반복합니다.
    4. 키보드 원격 조작 노드를 실행하여 지도에서 로봇의 정확한 위치를 파악합니다.  
        `$ roslaunch turtlebot3_teleop turtlebot3_teleop_key.launch`
    5. 로봇을 앞뒤로 조금 움직여 주변 환경 정보를 수집하고 작은 녹색 화살표로 표시되는 지도에서 TurtleBot3의 예상 위치를 좁힙니다.
    6. 탐색 중에 여러 노드에서 서로 다른 *cmd_vel* 값이 게시되는 것을 방지하기 위해 키보드 원격 조작 노드를 종료하려면 원격 조작 노드 터미널에 `Ctrl` + `C`를 입력하여 키보드 원격 조작 노드를 종료합니다.
3. Set Navigation Goal
    1. RViz 메뉴에서 `2D Nav Goal` 버튼을 클릭합니다.
    2. 지도를 클릭하여 로봇의 목적지를 설정하고 녹색 화살표를 로봇이 향할 방향으로 드래그합니다.
       * 녹색 화살표는 로봇의 목적지를 지정할 수 있는 마커입니다.
       * 화살표의 근원은 목적지의 `x`, `y` 좌표이며 각도 `θ`는 화살표의 방향에 따라 결정됩니다.
       * x, y, θ가 설정되면 TurtleBot3는 즉시 목적지로 이동하기 시작합니다.

## Simulation
Install Simulation Package

TurtleBot3 시뮬레이션 패키지를 사용하려면 전제 조건으로 `turtlebot3` 및 `turtlebot3_msgs` 패키지가 필요합니다. 이 필수 패키지가 없으면 시뮬레이션을 실행할 수 없습니다.  
필수 패키지 및 종속 패키지를 설치하지 않은 경우 [PC 설치](https://emanual.robotis.com/docs/en/platform/turtlebot3/quick-start/) 안내를 따르십시오.

```shell
$ cd ~/catkin_ws/src/
$ git clone -b noetic-devel https://github.com/ROBOTIS-GIT/turtlebot3_simulations.git
$ cd ~/catkin_ws && catkin_make
```

Launch Simulation World
터틀봇3에는 세 가지 시뮬레이션 환경이 준비되어 있습니다. 이 환경 중 하나를 선택하여 가제보를 실행하세요.  
_새 월드를 시작하기 전에 다른 시뮬레이션 월드를 완전히 종료해야 합니다._

1. Empty World  
    ```shell
    $ export TURTLEBOT3_MODEL=waffle_pi
    $ roslaunch turtlebot3_gazebo turtlebot3_empty_world.launch
    ```
2. TurtleBot3 World  
    ```shell
    $ export TURTLEBOT3_MODEL=waffle_pi
    $ roslaunch turtlebot3_gazebo turtlebot3_world.launch
    ```
3. TurtleBot3 House  
    ```shell
    $ export TURTLEBOT3_MODEL=waffle_pi
    $ roslaunch turtlebot3_gazebo turtlebot3_house.launch
    ```

NOTE: TurtleBot3 House를 처음 실행하는 경우, 네트워크 상태에 따라 지도를 다운로드하는 데 몇 분 이상 걸릴 수 있습니다.

Operate TurtleBot3

키보드로 터틀봇3를 원격 조종하기 위해서는 새 터미널 창에서 아래 명령어로 원격 조종 노드를 실행합니다.  
```shell
export TURTLEBOT3_MODEL=waffle_pi
roslaunch turtlebot3_teleop turtlebot3_teleop_key.launch
```

### SLAM Simulation
가제보 시뮬레이터에서 SLAM을 실행하면 가상세계에서 다양한 환경과 로봇 모델을 선택하거나 생성할 수 있습니다. 로봇을 불러오는 대신 시뮬레이션 환경을 준비하는 것 외에는 실제 터틀봇3를 이용한 SLAM 시뮬레이션과 매우 유사합니다.

1. Launch Simulation World  
    세 가지 Gazebo 환경이 준비되어 있지만 SLAM으로 맵을 만들려면 *TurtleBot3 World* 또는 **TurtleBot3 House**를 사용하는 것이 좋습니다.
2. Run SLAM Node  
    원격 PC에서 `Ctrl + Alt + T`로 새 터미널을 열고 SLAM 노드를 실행합니다. 기본적으로 Gmapping SLAM 방식이 사용됩니다.  
    ```shell
    $ export TURTLEBOT3_MODEL=waffle_pi
    $ roslaunch turtlebot3_slam turtlebot3_slam.launch slam_methods:=gmapping
    ```
3. Run Teleoperation Node  
    새 터미널:  
    ```shell
    $ export TURTLEBOT3_MODEL=waffle_pi
    $ roslaunch turtlebot3_teleop turtlebot3_teleop_key.launch
    ```
4. Save Map  
    지도가 성공적으로 생성되면 원격 PC에서 `Ctrl + Alt + T`를 사용하여 새 터미널을 열고 지도를 저장합니다.  
    `rosrun map_server map_saver -f ~/map`
