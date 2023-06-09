# JPA Transactional readOnly

JPA Transactional readOnly를 사용하는 이유, 더 깊게 생각해보기

### 1. Insert, Update, Delete를 막아줌?

@Transactional(readOnly = True) 가 설정되어있다고 하더라도 수정을 막을지는 확실할 수 없다. JPA는 DB 벤더에 상관없이 기능이 똑같이 수행하고, 벤더에 따라 오류로 처리될 수 도, 정상으로 처리될 수 도 있다.

### 2. DB 벤더마다 다른 효과?

Read Only 설정을 처리하는 방식은 DB 벤더마다 다르다고 했다. 특정 벤더에 따라 읽기 전용 설정이 데이터 안정성, 성능 향상을 야기하기도 한다.

Oracle의 경우, Read Only 트랜잭션을 이용할 경우 <b>이 트랜잭션이 시작되기 이전에 커밋된 데이터만 접근할 수 있으며, 트랜잭션 실행되는 동안 커밋되는 데이터는 결과에 반영되지 않는다</b>. 즉 해당 트랜잭션 내에서 일관된 데이터를 보장한다. 
 
MySql은 Oracle의 안정성에 더해 <b>트랜젝션 ID를 부여하지 않아도 돼</b>, 불필요한 오버헤드를 발생하지 않도록 하여 성능 이점을 볼 수 있다.

### 3. 그 외에도

테이블 락을 생략하거나 엔티티 매니저를 닫을 때 <b>더티 체킹을 건너 뛰는</b> 등의 최적화를 유발 할 수 있고,     

DB 구조 / 설정에 따라 해당 옵션을 통해 쿼리가 Master DB가 아닌, <b>읽기 전용의 Slave DB를 향하게</b> 하는 등의 설정이 가능해진다는 이점이 있다.
