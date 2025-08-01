2025-1 고급 백엔드 스터디 week7
================================

* # 4장 B-트리 구현
* ## 리밸런싱
    * 일부 B-트리 구현 시 분할과 병합 비용을 줄이기 위해 레벨 내에서 노드를 리밸런싱하거나, 분할 및 병합 작업 수행 전 상대적으로 빈 공간이 많은 노드로 원소를 이동한다.
        * 이 경우 리밸런싱 비용이 높아질 수도 있지만 노드의 점유율을 높이고 트리의 높이는 낮출 수 있다.
    
    * 노드 삽입과 삭제 시 로드 밸런싱 수행
        * 노드를 분할하는 대신 형제 노드로 일부 원소를 옮기고 삽입할 공간 확보
        * 노드 삭제 시엔 형제 노드와 병합하는 대신 노드가 절반 이상 찬 상태를 유지하도록 형제 노드에서 일부 원소를 가져온다.

    * B<sup>*</sup>-트리는 형제 노드가 모두 가득 찰 때까지 이웃 노드 간에 원소를 분산한다.
        * B<sup>*</sup>-트리 알고리즘은 노드를 절반이 비어 있는 두 개의 노드로 분할하지 않고, 두 노드를 2/3가 채워진 3개의 노드로 분할한다.
        * 이 방식으로 분할을 지연시켜 평균 점유율을 높일 수 있으나, 상태를 관리하고 균형을 맞추는 로직이 추가로 필요하다.
        * 높은 점유율로 인해 트리 높이가 낮아지고 순회 시 참조하는 페이지 수가 줄어 검색의 효율성을 높일 수 있다.

* ## 오른쪽 추가 기법
    * 많은 데이터베이스 시스템에선 자동 증가 값을 기본 인덱스 키로 사용한다. 모든 삽입이 인덱스 끝(가장 오른쪽 리프)에서만 발생하는 방식이다.
        * 대부분의 노드 분할 작업이 각 레벨 가장 오른쪽 노드에서 일어나게 된다.
        * PostgreSQL에선 이 방식을 `패스트패스(fastpath)`라고 부른다.
            * 삽입하는 키가 가장 오른쪽 페이지의 첫 번째 키보다 크고 가장 오른쪽 페이지에 새로운 키를 삽입할 공간이 충분하면, 탐색 과정을 건너뛰고 캐시된 페이지의 알맞은 위치에 키를 바로 삽입한다.
        
        * SQLite에선 `퀵밸런스(quickbalance)`라는 개념을 사용한다.
            * 빈 공간이 없는 가장 오른쪽 노드에 데이터 추가 시 균형을 다시 맞추거나 분할하는 대신, 오른쪽에 새로운 노드를 할당하고 부모 노드에 포인터를 추가한다.
        
        * ### 벌크 로딩
            * 정렬된 데이터를 벌크 로딩하거나 트리 재구성 시 우측 추가 알고리즘을 사용할 수 있다.
                * 트리를 하위 레벨부터 상향식으로 한 레벨씩 구성하거나, 상위 레벨 노드에 충분한 수의 포인터를 넣을 수 있을 만큼 하위 레벨의 노드가 추가됐을 때 상위 노드를 쓰는 방식으로 트리를 구성하여 노드의 분할과 병합을 피할 수 있다.

            * 미리 정렬된 데이터를 리프 레벨에 페이지 단위로 저장하는 방식으로 벌크 로딩을 구현할 수 있다.
                * 리프 페이지 생성 후, 리프 페이지의 첫 번째 키를 부모 노드에 복사하고 상위 레벨은 일반 B-트리 알고리즘을 사용해 구성한다.
    
    * ## 압축
        * 원시 데이터를 압축하지 않고 저장하면 상당한 저장 오버헤드가 발생할 수 있다. 따라서 대부분의 데이터베이스는 공간 절약을 위한 압축 알고리즘을 제공한다.
        * 압축 알고리즘에서 접근 속도와 압축률은 서로 상반된 관계이다.
            * 압축률이 높을수록 데이터 크기는 감소하고 한번에 더 많은 데이터를 읽을 수 있다.
        
        * 데이터는 다양한 단위로 압축할 수 있다.
            * 파일 전체 압축 시 압축률이 향상되지만 업데이트 시 파일 전체를 다시 압축해야 하므로 비효율적이다.
            * 페이지 단위 압축 시 파일 전체 압축 시 발생하는 문제를 해결할 수 있다.
                * 페이지는 다른 페이지와 독립적으로 압축 및 압축 해제할 수 있기 때문에 페이지 로딩 및 플러시와 같이 수행할 수 있다.
                * 일반적으로 데이터 전송은 블록 단위로 이루어지므로, 압축된 페이지가 블록의 극히 일부분을 차지하는 경우에는 실제 데이터보다 더 많은 바이트를 읽는 비효율적인 상황이 발생할 수 있다.
            * 데이터를 로우 단위 또는 칼럼 단위로 압축할 수 있다.
                * 이 경우 페이지 관리와 압축 작업을 묶어서 수행할 수 없다.
        
        * 압축 알고리즘은 대상 데이터셋과 목표에 따라 결과가 달라질 수 있다.
            * 압축 라이브러리를 선택할 때 중요한 비교 요소로는 메모리 오버헤드, 압축 성능, 압축 해제 성능, 압축률이 있다.
    
    * ## 정리와 유지
        * 슬롯 페이지는 설계상 추가적인 페이지 관리가 필요하다.
            * 내부 노드의 분할과 병합, 리프 레벨 노드에 대한 삽입 및 업데이트, 삭제 등이 계속 일어나면 단편화가 발생해 논리적 공간은 충분하지만 연속된 물리적 공간이 부족한 페이지가 생긴다.
        
        * B-트리는 루트 노드에서부터 탐색을 시작한다.
            * `라이브(주소 참조 가능, addressable)` 상태 : 탐색 경로를 따라 접근할 수 있는 데이터의 상태
            * `가비지(garbage)` : 참조할 수 없는 데이터
                * 어디에서도 참조하지 않고 읽을 수 없으므로 null값과 같다.
            
        * ### 업데이트와 삭제로 인한 단편화
            * #### 가비지 데이터가 존재하는 페이지를 컴팩션하는 방법
                * 삭제된 리프 레벨의 셀은 헤더에서 오프셋만 제거하고 실제 셀은 남겨둔다.
                    * 해당 셀은 더 이상 참조할 수 없으며 관련 데이터는 쿼리 대상에서 제외된다.
                * 페이지가 분할되면 페이지 일부는 더 이상 참조할 수 없으므로 해당 오프셋은 삭제된다.
                * `단편화` : 사용 가능한 바이트가 페이지에 흩어져 있는 현상
                    * 삭제괸 셀의 오프셋만 삭제하고 다른 셀을 재배치하거나, 공간 확보를 위해 물리적으로 셀을 삭제하지 않을 경우 발생할 수 있다.
                    * 단편화된 여러 조각들을 모아 연속된 공간 확보를 위해선 페이지를 재구성해야 한다.

        * ### 페이지 단편화
            * 공간 회수 및 페이지를 재구성하는 과정을 컴팩션, 정리, 또는 유지보수라고 한다.
                * 페이지에 사용 가능한 물리적 공간이 부족할 경우, 컴팩션과 쓰기를 동시에 수행할 수 있다.
                * 일반적으로 컴팩션은 페이지별로 가비지 컬렉션을 수행하고 데이터를 재작성하는 독립적인 비동기적 과정을 말한다.
            
            * 컴팩션은 데드 셀이 차지하는 공간을 회수하고 셀을 논리적 순서로 재정렬한다.
                * 재구성된 페이지는 파일에서의 위치가 변경될 수 있다.
                * 사용 중이지 않은 인메모리 페이지는 사용 가능 상태로 변경되고 페이지 캐시에 반환된다.
                * 디스크에 새로 할당된 페이지의 ID는 프리 페이지 목록(free page list 또는 freelist)에 추가한다. 이 목록은 사용 가능한 공간이 누락되지 않도록 노드 장애나 재부팅 시에도 유지되어야 한다.