2025-1 고급 백엔드 스터디 week10
================================

* # 5장 트랜잭션 처리와 복구
    * ## 동시성 제어
        * ### 낙관적 동시성 제어
            * 낙관적 동시성 제어는 트랜잭션 충돌이 거의 발생하지 않는다고 가정한다.
                * 잠금과 블로킹 트랜잭션을 사용하지 않고 결과를 커밋하기 전에 트랜잭션을 검증해 동시 수행 트랜잭션의 읽기/쓰기 충돌을 방지ㅣ하고 직렬화 가능성을 확인한다.
            
            * 통상적인 트랜잭션 수행 단계
                * `읽기 단계`
                    * 트랜잭션은 자신이 변경한 내용을 다른 트랜잭션에서 볼 수 없도록 개별 컨텍스트에서 트랜잭션 단계를 수행한다.
                    * 이 단계 후에는 모든 트랜잭션 존속성과 트랜잭션의 효과를 알 수 있다.
                
                * `검증 단계`
                    * 동시 수행 트랜잭션의 읽기와 쓰기 대상에서 직렬화 가능성을 보장하지 않는 충돌이 발생할 수 있는지 확인한다.
                    * 트랜잭션이 쿼리한 데이터가 최신이 아니거나 읽기 단계 중에 수정 및 커밋한 값을 다른 트랜잭션이 덮어쓴 경우 컨텍스트를 초기화하고 읽기 단계부터 다시 수행한다.
                    * 트랜잭션을 커밋해도 ACID 속성이 유지되는지 검증하는 단계이다.

                * `쓰기 단계`
                    * 검증 단계에서 충돌이 발견되지 않았다면 결과를 개별 컨텍스트에서 데이터베이스 상태로 커밋한다.
            
            * 검증은 이미 커밋된 트랜잭션(역방향) 또는 현재 검증 중인 트랜잭션(순방향)과의 충돌 여부를 확인하는 작업이다.
                * 트랜잭션의 검증과 쓰기 단계는 원자적으로 수행되어야 한다.
                * 트랜잭션을 검증하는 동안에는 다른 트랜잭션은 커밋될 수 없다.
                
            * 역방향 동시성 제어는 모든 T<sub>1</sub>과 T<sub>2</sub> 트랜잭션 쌍에 대해 다음 속성을 보장한다.
                * T<sub>1</sub>의 읽기 단계가 시작되기 전에 T<sub>1</sub>이 커밋하면 T<sub>2</sub>도 커밋할 수 있다.
                * T<sub>2</sub>의 쓰기 단계가 시작되기 전에 T<sub>1</sub>의 쓰기 대상과 T<sub>2</sub>의 읽기 대상이 겹치지 않는다면 T<sub>1</sub>이 쓴 값은 T<sub>2</sub>에서 참조하지 않는다는 뜻이다.
                * T<sub>1</sub>의 읽기 단계가 T<sub>2</sub>의 읽기 단계보다 먼저 완료되고 T<sub>2</sub>의 쓰기 대상과 T<sub>1</sub>의 읽기 또는 쓰기 대상과 겹치지 않는다면 두 트랜잭션은 서로 독립적인 데이터 레코드를 사용하기 때문에 모두 커밋이 허용된다.

            * 낙관적 동시성 제어는 일반적으로 검증이 성공적이고 트랜잭션을 재시도할 필요가 없는 경우 효율적이다.        
        
        * ### 다중 버전 동시성 제어(MVCC)
            * 여러 버전의 레코드를 저장하고 단조 증가하는 트랜잭션 ID 또는 타임스탬프로 식별해 데이터베이스 트랜잭션의 일관성을 보장하는 동시성 제어 방식이다.
                * 새로운 버전이 커밋될 때까지 이전 버전을 읽을 수 있기 때문에 비교적 간단한 조정 단계를 통해 동시 읽기 및 쓰기 작업을 수행할 수 있다.
            
            * MVCC는 커밋된 값과 커밋되지 않은 값을 구분한다.
                * 가장 마지막에 커밋된 값이 현재 값이다.
                * 트랜잭션 매니저는 한 번에 최대 하나의 커밋되지 않은 값이 존재하도록 제어한다.
            
            * 읽기 작업이 커밋되지 않은 값을 참조할 수 있는지에 대한 여부는 격리 수준에 따라 바뀔 수 있다.
                * MVCC는 잠금과 스케줄링, 2단계 잠금과 같은 충돌 알고리즘 또는 타임스탬프 순서화 알고리즘을 사용해 구현할 수 있다.

        * ### 비관적 동시성 제어
            * 비관적 동시성 제어는 낙관적 동시성 제어보다 더 보수적이다. 트랜잭션 수행 중에 충돌 발생 가능성을 확인하고 계속 수행하거나 중단 또는 취소한다.

            * `타임스탬프 순서화 알고리즘`은 각 트랜잭션에 타임스탬프를 설정하는 가장 단순한(무잠금 방식의) 비관적 동시성 제어 방식이다.
                * 트랜잭션 수행 여부는 더 높은 타임스탬프가 설정된 트랜잭션이 커밋됐는지 여부에 따라 결정된다.
                * 트랜잭션 매니저는 값별로 읽기와 쓰기를 수행한 동시 수행 트랜잭션의 정보를 `max_read_timestamp`와 `max_write_timestamp`에 저장한다.
                    * `max_write_timestamp`보다 낮은 타임스탬프가 설정된 트랜잭션이 값을 요청할 경우 이미 새로운 버전의 값이 존재하므로 이 작업을 허용하면 트랜잭션 순서를 위반하게 된다.
                    * 마찬가지로 타임스탬프가 `max_read_timestamp`보다 낮은 쓰기 작업은 뒤에 실행된 읽기 작업과 충돌한다.
                    * 하지만 타임스탬프가 `max_write_timestamp`보다 낮은 쓰기 작업이 쓴 값은 무시해도 되기 때문에 허용된다.
                    * 이런 규칙을 `토마스 기록 규칙(Thomas Write Rule)`이라고 한다.
        
        * ### 잠금 기반 동시성 제어
            * 잠금 기반 동시성 제어는 타임스탬프 순서화 기법과 같은 스케줄링 기반이 아닌, 데이터베이스 객체에 명시적으로 잠금을 성정하는 비관적 동시성 제어의 한 종류이다.

            * 잠금 기반 방식은 `경합 현상` 및 `확장성 문제`가 발생할 수 있다는 단점이 있다.

            * `2단계 잠금(2PL, Two-Phase Locking)`은 가장 보편적인 잠금 기법이다. 다음 두 단계로 구성된다.
                1. `확장 단계(growing/expanding phase)` : 필요한 잠금을 획득하고 유지한다.
                2. `축소 단계(shrinking phase)` : 획득한 잠금을 해제한다.

            * 트랜잭션은 단 하나의 잠금이라도 해제하면 더 이상 다른 잠금을 획득할 수 없다.
                * 2PL은 어떤 단계에서도 트랜잭션 수행을 제한하지 않는다.
            
            * #### 교착 상태
                * 잠금 프로토콜에서 트랜잭션은 데이터베이스 객체에 대한 잠금을 요청한다.
                    * 잠금을 바로 획득하지 못할 경우 다른 트랜잭션이 잠금을 해제할 때
                    까지 기다려야 한다.
                    * 여러 트랜잭션이 잠금을 획득하는 과정에서 서로 사용 중인 잠금을 해제하기를 기다리는 상태를 `교착 상태(deadlock)`라고 한다.

                * 교착 상태를 해결하는 가장 간단한 방법은 `타임아웃`을 설정해 오랫동안 끝나지 않는 트랜잭션은 교착 상태가 발생한 것으로 간주하고 중단하는 것이다.
                    * 보수적 2PL의 경우 시작 전에 모든 잠금을 획득하지 못한 트랜잭션은 중단된다.
                        * 이런 방식은 시스템의 동시성을 크게 저하시키기 때문에 대부분의 데이터베이스는 트랜잭션 매니저를 통해 교착 상태를 감지 및 방지한다.
                
                * 트랜잭션 사이의 교착 상태를 방지하기 위해서는 스케줄러가 필요하다. 래치의 경우 교착 상태 방지 알고리즘을 사용하지 않고 프로그래머가 직접 교착 상태가 발생하지 않도록 구현해야 한다.
            
            * #### 잠금
                * 같은 데이터 세그먼트를 수정하는 두 개의 트랜잭션을 동시에 수행할 때 논리적 일관성을 보장하기 위해선 서로의 중간 결과를 볼 수 없어야 한다.
                    * 한편 동일한 트랜잭션에서 실행되는 스레드는 동일한 데이터베이스 상태에 대해 작업하고 서로의 결과에 접근할 수 있어야 한다.
                
                * 잠금은 동시 수행 트랜잭션을 격리 및 스케줄링하고 데이터베이스의 상태를 관리하는 데 사용된다.
                    * 잠금은 존재 여부와 상관없이 특정 키 또는 특정 범위의 키를 보호한다.
                
                * 잠금은 래치보다 무겁고 트랜잭션이 수행되는 동안 유지된다.
            
            * #### 래치
                * 래치(latch)는 물리적 구조를 보호한다.
                    * 래치는 물리적 트리 구조(페이지 및 트리 구조)를 보호하며 페이지에 대해 요청할 수 있다.
                    * 특정 페이지에 동시에 접근하기 위해서는 반드시 래치를 획득해야 한다.
                
                * 리프 노드 수정 시 B-트리의 상위 레벨도 변경될 수 있기 때문에 여러 레벨에 대한 래치가 필요할 수 있다.

                * 소스 노드와 대상 노드 모두에 데이터가 존재하거나 데이터가 아직 상위 노드로 전파되지 않은 불완전한 쓰기 또는 노드 분할은 수행 중인 트랜잭션에서 볼 수 없도록 해서 상태의 일관성을 유지해야 한다.

                * 동시 수행 작업의 분류
                    * `동시 읽기` : 여러 스레드가 같은 페이지를 요청하고 수정하지 않는다.
                    * `동시 업데이트` : 여러 스레드가 같은 페이지를 수정한다.
                    * `쓰기 중 읽기` : 스레드가 페이지를 수정하는 동안 다른 페이지가 같은 페이지를 읽는다.

            * #### 리더-라이터 잠금
                * 가장 간단한 래치 구현 방식은 요청하는 스레드에 배타적 읽기/쓰기를 허용하는 것이다.
                    * 대부분의 경우 모든 프로세스를 꼭 서로 분리할 필요는 없다. 여러 라이터(writer)가 동시에 겹치지 않고 리더(reader)와 라이터가 겹치지 않도록 하면 된다.
                
                * 리더-라이터(RW, Readers-Writer) 잠금은 여러 리더가 동시에 같은 객체를 읽는 것을 허용한다. 반면 라이터는 객체를 독점해야 한다.
                    * 오직 리더만이 `공유 잠금(shared lock)`을 소유할 수 있고 모든 다른 리더와 라이터 조합은 배타적 잠금을 요청해야 한다.
                
                * 동일한 페이지에 동시에 접근하는 읽기 작업은 페이지 캐시가 디스크에서 같은 페이지를 반복해서 페이징하는 것만 방지할 수 있다면 동기화가 필요 없고 공유 잠금 모드에서 동시에 안전하게 읽을 수 있다. 쓰기 작업은 다른 동시 수행 읽기 및 쓰기 작업과 분리돼야 한다.
            
            * #### 래치 크래빙
                * 가장 단순한 래치 획득 방식은 루트에서부터 대상 리프 노드 사이의 모든 래치를 획득하는 것이다. 이 방식은 동시성 병목 현상이 발생할 수 있다.
                    * 이를 해결하기 위해 래치를 소유하는 시간을 최소화해야 하며, 래치 크래빙(crabbing, 또는 래치 결합) 기법을 사용할 수 있다.

                * 래치 크래빙 방식은 래치를 최대한 짧게 소유하고 작업을 수행하는데 래치가 더 이상 필요하지 않을 경우 바로 해제한다.
                    * 자식 노드를 찾으면 즉시 해당 래치를 획득하고 부모 노드의 래치는 해제한다.
                    * 노드 삽입으로 인해 부모 노드를 포함한 트리 구조가 변경되지 않을 것이 확실할 경우 부모 레벨의 래치는 바로 해제한다. 즉, 자식 노드가 가득 찬 상태가 아니라면 부모 레벨의 래치는 해제한다.
                    * 자식 페이지가 아직 페이지 캐시에 페이징되지 않은 경우 해당 페이지에 대한 래치를 획득하거나 경합을 피하기 위해 부모 래치를 해제하고 페이징이 완료되면 루트에서부터 다시 탐색한다.
            
            * #### B<sup>link</sup>-트리
                * B<sup>link</sup>-트리는 B<sup>*</sup>트리에 하이 키와 형제 링크 포인터를 추가한 자료 구조다.
                    * B<sup>link</sup>-트리의 루트 노드를 제외한 모든 노드에는 포인터가 두 개씩 있다.
                        * 부모가 자식 노드를 가리키는 자식 포인터와 같은 레벨의 형제 노드를 가리키는 `형제 링크 포인터`가 있다.
                
                * B<sup>link</sup>-트리에는 중간 분할(half-split) 상태가 존재한다.
                    * 노드를 가리키는 형제 포인터는 있지만 아직 부모 노드에서 참조하는 자식 포인터가 없는 상태를 나타낸다.
                    * 중간 분할 상태는 노드의 하이 키를 통해 확인할 수 있다.
                    * 만약 검색 키가 노드의 하이 키보다 클 경우 룩업 알고리즘은 트리의 구조가 현재 변경 중이라고 간주하여 형제 링크를 따라 계속해서 탐색한다.
                
                * 이런 방식은 자식 노드가 분할되더라도 부모 노드에 대한 잠금을 유지할 필요가 없다는 것이 장점이다.
                    * 형제 링크를 통해 새로운 노드에 접근할 수 있으므로 부모 노드는 천천히 업데이트해도 정확성이 보장된다.
                
                * 이런 방식은 부모 노드에서 바로 자식노드를 참조하는 것보다 덜 효율적이고 접근해야 하는 페이지 수도 증가하지만, 루트-리프 탐색 시 동시 접근을 단순화할 수 있다.
                    * 일반적으로 노드 분할은 자주 발생하지 않고 B-트리의 크기도 좀처럼 줄어들지 않기 때문에, 중간 분할 상태는 예외적으로 발생하고 비용도 미미하다.
                    * 경합이 감소하고 분할 중 부모 노드에 대한 잠금을 유지하지 않아도 되기 때문에 구조 변경 시 필요한 잠금 수가 줄어드는 장점이 있다.
                    * 트리의 구조 변경과 읽기를 동시에 수행할 수 있으며, 부모 노드를 동시에 수정하려는 시도로 인해 발생하는 교착 상태를 방지할 수 있다.