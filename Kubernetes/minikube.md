minikube 실습
-----

minikube는 로컬에서 kubernetes환경을 간단하게 구성할수 있기 때문에 많이 사용이 된다.

사용법도 간단해서 kubenetes를 학습하기 위한 최적의 방법을 제공해 준다.

[Windows 설치파일](https://github.com/kubernetes/minikube/releases/latest/download/minikube-installer.exe "minikube-installer.exe")


버전확인
<pre>
<code>
minikube version
</code>
</pre>

가상머신 시작 
<pre>
<code>
minikube start                                  // 기본
minikube start --driver=docker                  // docker desktop 이용
minikube start --driver=hyperv                  // hyperv 이용
minikube start --driver=virtualbox              // virtual box 이용
minikube start --kubernetes-version=v1.20.0     // kubenetes 버전 지정
</code>
</pre>

상태확인
<pre>
<code>
minikube status
</code>
</pre>

정지
<pre>
<code>
minikube stop
</code>
</pre>

삭제
<pre>
<code>
minikube delete
</code>
</pre>

ssh 접속
<pre>
<code>
minikube ssh
</code>
</pre>

ip 확인
<pre>
<code>
minikube ip
</code>
</pre>

다중 노드
<pre>
<code>
minikube start
minikube start -n 3     // 다중 노드
</code>
</pre>

프로필
<pre>
<code>
minikube start                  // minikube profile로 생성
minikube start -p helloworld    // helloworld profile로 생성
</code>
</pre>

profile 목록
<pre>
<code>
minikube profile list
</code>
</pre>

현재 profile 확인
<pre>
<code>
minikube profile
</code>
</pre>

profile로 변경
<pre>
<code>
minikube profile helloworld     // helloworld profile로 변경
minikube profile minikube       // minikube profile로 변경
</code>
</pre>

가상머신 제거
<pre>
<code>
minikube delete                 // 현재 profile 가상머신 제거
minikube delete --all           // 전체 제거
</code>
</pre>

대쉬보드
<pre>
<code>
minikube dashboard
</code>
</pre>

일시정지
<pre>
<code>
minikube pause                  // 일시정지
minikube unpause                // 일시정지 해제
</code>
</pre>

일시정지
<pre>
<code>
minikube config set memory 16384        // 메모리 설정
minikube config unset memory            // 메모리 설정 초기화
minikube config view                    // 설정 보기
</code>
</pre>

설정 옵션
| 값 | 설명 |
|---|:---:|
| defaults |    Lists all valid default values for PROPERTY_NAME |
| get |         Gets the value of PROPERTY_NAME from the minikube config file |
| set |        Sets an individual value in a minikube config file |
| unset |      unsets an individual value in a minikube config file |
| view |       Display values currently set in the minikube config file |

설정 가능한 항목
 * driver
 * vm-driver
 * container-runtime
 * feature-gates
 * v
 * cpus
 * disk-size
 * host-only-cidr
 * memory
 * log_dir
 * kubernetes-version
 * iso-url
 * WantUpdateNotification
 * WantBetaUpdateNotification
 * ReminderWaitPeriodInHours
 * WantNoneDriverWarning
 * profile
 * bootstrapper
 * insecure-registry
 * hyperv-virtual-switch
 * disable-driver-mounts
 * cache
 * EmbedCerts
 * native-ssh

[참고자료] https://minikube.sigs.k8s.io/docs/start/

