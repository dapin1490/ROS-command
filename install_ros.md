<h1>ROS 1, 2 설치 팁</h1>

- [ROS1 noetic](#ros1-noetic)
  - [PC setup](#pc-setup)
    - [패키지 설치](#패키지-설치)
    - [네트워크 설정](#네트워크-설정)
  - [SBC setup](#sbc-setup)
    - [SD카드 굽기](#sd카드-굽기)
    - [SD카드에 와이파이 정보 입력](#sd카드에-와이파이-정보-입력)
    - [ROS 네트워크 설정](#ros-네트워크-설정)
    - [추가 드라이버와 패키지 설치](#추가-드라이버와-패키지-설치)
  - [OpenCR setup](#opencr-setup)
- [ROS2 turtlesim](#ros2-turtlesim)
  - [필수 설치](#필수-설치)
  - [선택 설치](#선택-설치)
    - [vscode](#vscode)
  - [ROS2 삭제](#ros2-삭제)

# ROS1 noetic
## PC setup
* Ubuntu 20.04 LTS 버전의 리눅스 PC(가상머신)에 설치한다

### 패키지 설치
다음 스크립트를 한 줄씩 실행(`$` 하나에 한 줄)

```shell
$ sudo apt update
$ sudo apt upgrade
$ wget https://raw.githubusercontent.com/ROBOTIS-GIT/robotis_tools/master/install_ros_noetic.sh
$ chmod 755 ./install_ros_noetic.sh
$ bash ./install_ros_noetic.sh

$ sudo apt-get install ros-noetic-joy ros-noetic-teleop-twist-joy \
  ros-noetic-teleop-twist-keyboard ros-noetic-laser-proc \
  ros-noetic-rgbd-launch ros-noetic-rosserial-arduino \
  ros-noetic-rosserial-python ros-noetic-rosserial-client \
  ros-noetic-rosserial-msgs ros-noetic-amcl ros-noetic-map-server \
  ros-noetic-move-base ros-noetic-urdf ros-noetic-xacro \
  ros-noetic-compressed-image-transport ros-noetic-rqt* ros-noetic-rviz \
  ros-noetic-gmapping ros-noetic-navigation ros-noetic-interactive-markers

$ sudo apt install ros-noetic-dynamixel-sdk
$ sudo apt install ros-noetic-turtlebot3-msgs
$ sudo apt install ros-noetic-turtlebot3
```

### 네트워크 설정
1. `ifconfig`로 현재 PC의 IP 주소 확인 -> `192.168.0.xx`와 같이 쓰인 것을 찾으면 됨
2. `nano ~/.bashrc`로 .bashrc 파일 열기
3. 스크롤 맨 아래로 내려 `ROS_MASTER_URI`와 `ROS_HOSTNAME` 둘 다 `localhost`를 지우고 1번에서 확인한 IP 주소를 쓰면 됨. 중괄호가 있다면 그것도 지울 것.
4. `Ctrl+s`, `Ctrl+x` 입력하여 저장 후 터미널로 나가고, `source ~/.bashrc` 입력하여 변경사항 적용하기

## SBC setup
* **"remote PC"를 제외한 나머지는 전부 로봇이라고 생각하는 편이 대체로 맞다**
* SBC도 로봇이다.

### SD카드 굽기
* 다음 과정은 리눅스 PC에서 진행하는 것을 권장하나, 여의치 않을 경우 윈도우에서도 할 수 있다. 아래 과정은 리눅스를 기준으로 서술되었으며 윈도우는 다소 다를 수 있다.
* 이 과정을 수행하는 PC에는 최소 14기가 이상의 여유 공간이 필요하다.

<!--  -->

1. 16기가 이상의 SD 카드 준비
2. [Download Raspberry Pi 4B (2GB or 4GB) ROS Noetic image](https://www.robotis.com/service/download.php?no=2066&_gl=1*1tod3xm*_gcl_au*MTA1NTQxOTUwOS4xNzMyMTYxODU2) 이 링크에서 라즈베리파이 이미지 파일 다운로드하기
    * 다운로드 후 압축 해제. 해제된 파일 더블클릭 X. exe처럼 직접 사용하는 파일이 아님.
3. 두 가지 방법으로 SD카드를 구울 수 있다. **둘 중 하나만 하면 된다.**
    1. 라즈베리파이 이미저 사용하기
        1. 만약 설치가 필요할 경우 터미널에 `sudo snap install rpi-imager` 입력하여 설치
        2. 터미널에 `rpi-imager` 입력하여 라즈베리파이 이미저 실행
        3. `CHOOSE OS` 클릭
        4. `Use custom` 클릭 후 2에서 다운받고 압축 해제한 파일(`~~~.img` 라는 이름) 선택
        5. `CHOOSE STORAGE` 클릭 후 SD카드 선택
        6. `WRITE` 클릭 후 끝날 때까지 기다리기
        * 이미지로 보기  
            ![rpi_imager.gif](https://emanual.robotis.com/assets/images/platform/turtlebot3/setup/rpi_imager.gif)
    2. Disks 사용하기
        1. 리눅스에서 Disks 실행하기
        2. 왼쪽 패널에서 SD카드 선택
        3. 창 오른쪽 위에 있는 점 세 개 메뉴 눌러 `Restore Disk Image` 선택
        4. `image to restore` 메뉴에 2에서 다운받고 압축 해제한 파일(`~~~.img` 라는 이름) 선택해 넣기
        5. `Start Restoring...`, `Restore` 누르기
        * 이미지로 보기  
            ![disks.gif](https://emanual.robotis.com/assets/images/platform/turtlebot3/setup/disks.gif)
4. 파티션 복구하기: SD카드를 빠르게 굽기 위해 파티션을 축소한 상태로 구워진다. 축소된 파티션을 늘리는 과정임.
    1. GParted 실행. 없으면 설치하기.
    2. 화면 오른쪽 위에서 SD카드 선택
    3. 노란색 파티션 우클릭
    4. `Resize/Move` 클릭
    5. 직후 나타나는 창에서 오른쪽 슬라이드바를 맨끝까지 밀기
    6. `Resize/Move` 클릭
    7. 창 위쪽 리본메뉴에서 초록색 체크 마크로 표시된 `Apply All Operations` 버튼 클릭
    * 이미지로 보기  
        ![gparted.gif](https://emanual.robotis.com/assets/images/platform/turtlebot3/setup/gparted.gif)

### SD카드에 와이파이 정보 입력
1. 위의 SD카드 굽기를 완료한 직후엔 SD카드가 unmount된 상태일 것이므로, 가상머신을 재부팅하거나 SD를 뺐다가 다시 꽂아 PC에 연결한다.
2. 터미널에 다음과 같이 입력
    1. `cd /media/$USER/writable/etc/netplan`
    2. `sudo nano 50-cloud-init.yaml`
3. `WIFI_SSID`를 지우고 연결해야 할 와이파이 이름 입력
4. `WIFI_PASSWORD`를 지우고 해당 와이파이의 비밀번호 입력
5. `Ctrl+s`, `Ctrl+x` 입력하여 저장 후 터미널로 나가기

### ROS 네트워크 설정
* **여기서부터는 로봇에 모니터와 키보드를 연결하거나, ssh로 로봇에 원격 접속한 상태로 로봇의 터미널(이름이 `ubuntu`로 나옴)에서 진행해야 한다.**
* ssh 접속 방법은 [commands_ros1.md](commands_ros1.md#팁)를 참고한다.

1. `ifconfig`로 로봇의 IP 주소를 확인한다. 이미 알고 있다면 생략해도 된다.
2. 터미널에 `nano ~/.bashrc` 입력하여 `.bashrc` 파일 열기
3. 다음과 같이 생긴 부분을 찾는다(보통 맨 밑에 있음)  
    `export ROS_MASTER_URI=http://{IP_ADDRESS_OF_REMOTE_PC}:11311`  
    `export ROS_HOSTNAME={IP_ADDRESS_OF_RASPBERRY_PI_3}`  
4. `ROS_MASTER_URI`에 있는 `{IP_ADDRESS_OF_REMOTE_PC}`를 지우고, 로봇과 연결한(연결할) PC의 현재 IP 주소를 쓴다
5. `ROS_HOSTNAME`에 있는 `{IP_ADDRESS_OF_RASPBERRY_PI_3}`를 지우고, 로봇의 현재 IP 주소를 쓴다.
6. `Ctrl+s`, `Ctrl+x` 입력하여 저장 후 터미널로 나가고, `source ~/.bashrc` 입력하여 변경사항 적용하기

### 추가 드라이버와 패키지 설치
* 로봇 터미널에서 실행해야 한다

다음 명령어를 한 줄씩(`$` 하나에 한 줄) 터미널에 입력한다.

```shell
$ sudo apt update
$ sudo apt install libudev-dev
$ cd ~/catkin_ws/src
$ git clone -b noetic https://github.com/ROBOTIS-GIT/ld08_driver.git # 이 명령어 실행 후 디렉토리가 비어있지 않다는 오류가 발생한다면 무시해도 좋다
$ cd ~/catkin_ws/src/turtlebot3 && git pull
$ rm -r turtlebot3_description/ turtlebot3_teleop/ turtlebot3_navigation/ turtlebot3_slam/ turtlebot3_example/ # 이 명령어 실행 후 삭제할 파일 또는 폴더가 없다는 오류가 발생한다면 무시해도 좋다
$ cd ~/catkin_ws && catkin_make

$ echo 'export LDS_MODEL=LDS-02' >> ~/.bashrc
$ source ~/.bashrc
```

## OpenCR setup
* **로봇 터미널에서 실행한다**

다음 명령어를 한 줄씩(`$` 하나에 한 줄) 터미널에 입력한다.

```shell
$ sudo dpkg --add-architecture armhf
$ sudo apt-get update
$ sudo apt-get install libc6:armhf

$ export OPENCR_PORT=/dev/ttyACM0
$ export OPENCR_MODEL=burger_noetic
$ rm -rf ./opencr_update.tar.bz2

$ wget https://github.com/ROBOTIS-GIT/OpenCR-Binaries/raw/master/turtlebot3/ROS1/latest/opencr_update.tar.bz2
$ tar -xvf opencr_update.tar.bz2

$ cd ./opencr_update
$ ./update.sh $OPENCR_PORT $OPENCR_MODEL.opencr
```

모두 성공하면 다음과 같은 출력으로 끝난다

![OpenCR setup end](https://emanual.robotis.com/assets/images/platform/turtlebot3/opencr/shell01.png)


---

# ROS2 turtlesim
* 모든 자료는 [000 로봇 운영체제 ROS 강좌 목차](https://cafe.naver.com/openrt/24070)를 참고한다

<!--  -->

* 권장 OS: Ubunru 20.04 LTS

## 필수 설치
다음 명령어를 한 줄씩(`$` 하나에 한 줄) 터미널에 입력한다.

```shell
$ sudo apt update && sudo apt install locales
$ sudo locale-gen en_US en_US.UTF-8
$ sudo update-locale LC_ALL=en_US.UTF-8 LANG=en_US.UTF-8
$ export LANG=en_US.UTF-8

$ sudo apt update && sudo apt install curl gnupg2 lsb-release
$ sudo curl -sSL https://raw.githubusercontent.com/ros/rosdistro/master/ros.key  -o /usr/share/keyrings/ros-archive-keyring.gpg
$ echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/ros-archive-keyring.gpg] http://packages.ros.org/ros2/ubuntu focal main" | sudo tee /etc/apt/sources.list.d/ros2.list > /dev/null

$ sudo apt update
$ sudo apt install ros-foxy-desktop ros-foxy-rmw-fastrtps* ros-foxy-rmw-cyclonedds*
```

위의 설치가 정상적이라면 다음 두 블록을 두 개의 터미널에서 실행했을 때 출력문이 서로 유사해야 한다.

```shell
$ source /opt/ros/foxy/setup.bash
$ ros2 run demo_nodes_cpp talker
# 이 아래는 예시 출력이다
[INFO] [1612912263.574031946] [talker]: Publishing: 'Hello World: 1'
[INFO] [1612912264.574010597] [talker]: Publishing: 'Hello World: 2'
...
```

```shell
$ source /opt/ros/foxy/setup.bash
$ ros2 run demo_nodes_py listener
# 이 아래는 예시 출력이다
[INFO] [1612912265.593335793] [listener]: I heard: [Hello World: 1]
[INFO] [1612912266.576514520] [listener]: I heard: [Hello World: 2]
...
```

이어서 설치한다. 다음 명령어를 한 줄씩(`$` 하나에 한 줄) 터미널에 입력한다.

```shell
$ sudo apt update && sudo apt install -y \
  build-essential \
  cmake \
  git \
  libbullet-dev \
  python3-colcon-common-extensions \
  python3-flake8 \
  python3-pip \
  python3-pytest-cov \
  python3-rosdep \
  python3-setuptools \
  python3-vcstool \
  wget

$ python3 -m pip install -U \
  argcomplete \
  flake8-blind-except \
  flake8-builtins \
  flake8-class-newline \
  flake8-comprehensions \
  flake8-deprecated \
  flake8-docstrings \
  flake8-import-order \
  flake8-quotes \
  pytest-repeat \
  pytest-rerunfailures \
  pytest

$ sudo apt install --no-install-recommends -y \
  libasio-dev \
  libtinyxml2-dev \
  libcunit1-dev
```

설치가 정상적이라면 다음 명령어를 실행했을 때 0개의 패키지를 빌드했다는 출력문이 나와야 한다.

```shell
$ source /opt/ros/foxy/setup.bash
$ mkdir -p ~/robot_ws/src
$ cd ~/robot_ws/
$ colcon build --symlink-install
```

다음으로 `.bashrc`에 명령어를 설정한다

터미널에 `gedit ~/.bashrc`를 입력하여 메모장을 연다. 해당 메모장의 맨 아래에 빈 줄을 몇 줄 넣고 아래 내용을 복사해넣는다.

```shell
source /opt/ros/foxy/setup.bash
source ~/robot_ws/install/local_setup.bash

source /usr/share/colcon_argcomplete/hook/colcon-argcomplete.bash
source /usr/share/vcstool-completion/vcs.bash
source /usr/share/colcon_cd/function/colcon_cd.sh
export _colcon_cd_root=~/robot_ws

export ROS_DOMAIN_ID=7
export ROS_NAMESPACE=robot1

export RMW_IMPLEMENTATION=rmw_fastrtps_cpp
# export RMW_IMPLEMENTATION=rmw_connext_cpp
# export RMW_IMPLEMENTATION=rmw_cyclonedds_cpp
# export RMW_IMPLEMENTATION=rmw_gurumdds_cpp

# export RCUTILS_CONSOLE_OUTPUT_FORMAT='[{severity} {time}] [{name}]: {message} ({function_name}() at {file_name}:{line_number})'
export RCUTILS_CONSOLE_OUTPUT_FORMAT='[{severity}]: {message}'
export RCUTILS_COLORIZED_OUTPUT=1
export RCUTILS_LOGGING_USE_STDOUT=0
export RCUTILS_LOGGING_BUFFERED_STREAM=1

alias cw='cd ~/robot_ws'
alias cs='cd ~/robot_ws/src'
alias ccd='colcon_cd'

alias cb='cd ~/robot_ws && colcon build --symlink-install'
alias cbs='colcon build --symlink-install'
alias cbp='colcon build --symlink-install --packages-select'
alias cbu='colcon build --symlink-install --packages-up-to'
alias ct='colcon test'
alias ctp='colcon test --packages-select'
alias ctr='colcon test-result'

alias rt='ros2 topic list'
alias re='ros2 topic echo'
alias rn='ros2 node list'

alias killgazebo='killall -9 gazebo & killall -9 gzserver  & killall -9 gzclient'

alias af='ament_flake8'
alias ac='ament_cpplint'

alias testpub='ros2 run demo_nodes_cpp talker'
alias testsub='ros2 run demo_nodes_cpp listener'
alias testpubimg='ros2 run image_tools cam2image'
alias testsubimg='ros2 run image_tools showimage'
```

## 선택 설치
### vscode
다양한 기능을 지원할 수 있는 코드 편집기이다.

터미널에 `sudo snap install --classic code`를 입력하면 설치된다.

권장 확장 프로그램은 [001 ROS 2 개발 환경 구축](https://cafe.naver.com/openrt/25288)에서 참고하기. 안 해도 무방하다.

## ROS2 삭제
터미널에 `sudo apt remove ros-foxy-* && sudo apt autoremove` 입력하기

PC에 설치된 ROS2 패키지는 모두 삭제되지만 위에서 수정한 `.bashrc` 파일은 자동으로 수정되지 않기 때문에 설치 과정에서 입력한 만큼 직접 지워줘야 한다.