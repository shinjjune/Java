#### 1. e.printStackTrace()
 : exception 처리를 하면 자동적으로 생기는 함수
 : 처음 호출한데서부터 에러 발생한 끝까지 들어간 내용 그대로 보여준다.


#### 2. insert batch (oracle)
```
private static double insertBatchFileToDB(List<Object> list) throws Exception{
    long start = System.currentTimeMillis();    // 작동 시간 측정용
 
    // ojdbc6.jar 사용. import는 oracle.jdbc.*  
    OracleConnection conn = null;        // 통상 Connection으로 선언
    OraclePreparedStatement ps = null;    // 통상 PreparedStatement로 선언
 
    StringBuffer sql = new StringBuffer("INSERT INTO TB_TEST (")
                    .append("COL1")
                    .append(", COL2")
                    .append(", COL3")
                    .append(", COL4")
                    .append(", COL5")
                    .append(", COL6")
                    .append(", COL7")
                    .append(", COL8")
                    .append(") VALUES(?, ?, ?, ?, ?, ?, ?, ?)");
 
    try{
        conn = (OracleConnection) getOrclConn();    // Connection 생성 시 OracleConnection으로 형변환
        conn.setAutoCommit(false);                    // 자동 commit 끔
 
        // prepareStatement를 OraclePreparedStatement으로 형변환
        ps = (OraclePreparedStatement) conn.prepareStatement(sql.toString());
 
        int cnt = 0;    // row Count
 
        for(Object obj : list){
            ps.setString(1, obj.getCol1());
            ps.setString(2, obj.getCol2());
            // 생략 ...
            ps.setBigDecimal(8, obj.getCol8());
 
            ps.addBatch();    // OraclePreparedStatement에 batch로 완성된 SQL 추가
            ps.clearParameters();    // OraclePreparedStatement에 지정된 Parameter값 초기화
            
            cnt++;
            if((cnt % 10000) == 0){    // batch에 누적된 건수가 1만건
                cnt = 0;
                ps.executeBatch();    // 누적된 batch 실행
                ps.clearBatch();    // 누적된 batch 초기화
                conn.commit();        // Commit하여 적용
            }
        }
 
        // 최종적으로 누적된 채 남은 batch 작업(위의 if문) 실행
        ps.executeBatch();
        ps.clearBatch();
        conn.commit();
 
    }catch(Exception e){
        e.printStackTrace();
    }finally{
        ps.close();
        conn.close();
    }
 
    long end = System.currentTimeMillis();    // 작동시간 측정용
 
    return (end - start) / 1000.0;    // insertBatchFileToDB 실행에 걸린 시간 
}
```
#### 3. indexOf()
: "문자열".inddexOf("찾을 문자")
 -> 문자열에서 원하는 문자여을 검색하여 찾거나 아니면 배열에서 원하는 특정 배열값의 존재여부 등을 확인.
 ```
 var text = "456789";
var findStr = "123"; // 123이 있는지 찾아보기

if (text.indexOf(findStr) != -1) {
  alert("Find!");
}
else {
  alert("Not Found!!");
}
 ```

!-1 의 의미: -1이란 값이 존재하지 않음을 의미

#### 4. 시퀀스??
- 유일한 값을 생성해주는 오라클 객체
- 일련번호, 자동증가 값을 생성
- 시퀀스는 테이블과 별개로 동작(독립적)
- 모든 DBMS에서 사용하는 것은 아님
- 시퀀스는 메모리에 Cache하여 성능을 향상 시킬 수 있음

###### 보통은 테이블에서 기본키(P.K)를 생성하기 위해 사용하곤 함.
- NEXTVAL: 시퀀스의 값을 증가
- CURRVAL: 현재 시퀀스 값

#### 5. PL/SQL tip
- 'for update'

select * from [table] for update;  ->  데이터 수정가능


