# 진단검사 사이트

## 홈화면
![image](https://github.com/hwan06/health_examination_site/assets/114748934/8554ffb3-5945-4b5f-af0b-bf185f181026)   

## 환자조회화면
![image](https://github.com/hwan06/health_examination_site/assets/114748934/3be6f75e-9ce7-4757-9944-b71fe07c3e5b)   
### DB에서 형식에 맞춰 데이터 가져오는 sql코드
```sql
<%
	Connection conn = DBconnect.getConnection();
	String sql = "select p_no, p_name, substr(p_birth, 1, 4)||'년'||substr(p_birth, 5, 2)||'월'||substr(p_birth, 7, 2)||'일' as p_birth, "
	+ " case when p_gender = 'M' then '남' else '여' end as p_gender, p_tel1||'-'|| p_tel2||'-'||p_tel3 as phone, "
	+ " case when p_city = '10' then '서울' when p_city = '20' then '경기' when p_city = '30' then '강원' else '대구' end as p_city "
	+ " from tbl_patient_202004";
	PreparedStatement ps = conn.prepareStatement(sql);
	ResultSet rs = ps.executeQuery();
%>
```
### 섭틀릿을 이용해 while문을 작성하여 테이블에 데이터 출력
```java
<table class="table" style="text-align: center; width: 750px">
			<tr>
				<th>환자번호</th>
				<th>환자성명</th>
				<th>생년월일</th>
				<th>성별</th>
				<th>전화번호</th>
				<th>지역</th>
			</tr>
			<%while(rs.next()) { %>
			<tr>
				<td><%=rs.getString("p_no")%></td>
				<td><%=rs.getString("p_name")%></td>
				<td><%=rs.getString("p_birth")%></td>
				<td><%=rs.getString("p_gender")%></td>
				<td><%=rs.getString("phone")%></td>
				<td><%=rs.getString("p_city")%></td>
			</tr>
			<%} %>
		</table>
```
## 검사결과입력 화면
![image](https://github.com/hwan06/health_examination_site/assets/114748934/7afe7e6e-8a2d-4bcc-9479-0c5b81cd2c39)   
### form태그를 이용하여 유효성검사와 _p페이지 호출
```java
<form name="data" method="post" action="input_p.jsp" onsubmit="return check()">
```
### 모든 값을 입력했는지 유효성검사하는 코드
```sql
<script type="text/javascript">
	function check() {
		if (!document.data.num.value) {
			alert("환자번호를 입력해주세요.");
			data.num.focus();
			return false;
		}else if (!document.data.c_code.value) {
			alert("검사코드를 선택해주세요.");
			data.c_code.focus();
			return false;
		}else if (!document.data.startdate.value) {
			alert("검사시작일지를 입력해주세요.");
			data.startdate.focus();
			return false;
		}else if (!document.data.status[0].checked && !document.data.status[1].checked) {
			alert("검사상태를 선택해주세요.");
			return false;
		}else if (!document.data.cleardate.value) {
			alert("검사완료일자를 입력해주세요.");
			return false;
		}else if (!document.data.result[0].checked && !document.data.result[1].checked && !document.data.result[2].checked) {
			alert("검사결과를 선택해주세요.");
			return false;
		}
		return true;
		alert("등록되었습니다.");
	}
</script>
```
### _p페이지에서 데이터 삽입
```sql
<%@ page import = "java.sql.*" %>
<%@ page import = "DB.DBconnect" %>
<%
	Connection conn = DBconnect.getConnection();
	String sql = "insert into tbl_result_202004 values(?, ?, ?, ?, ?, ?)";
	PreparedStatement ps = conn.prepareStatement(sql);
	ps.setString(1, request.getParameter("num"));
	ps.setString(2, request.getParameter("c_code"));
	ps.setString(3, request.getParameter("startdate"));
	ps.setString(4, request.getParameter("status"));
	ps.setString(5, request.getParameter("cleardate"));
	ps.setString(6, request.getParameter("result"));
	ps.executeUpdate();
%>
```
## 검사결과조회 화면
![image](https://github.com/hwan06/health_examination_site/assets/114748934/593bfe34-5523-40fb-9eff-c61f8aee396e)   
### DB에서 형식에 맞춰 데이터 가져오기
```sql
<%@ page import = "java.sql.*" %>
<%@ page import = "DB.DBconnect" %>
<%
	Connection conn = DBconnect.getConnection();
	String sql = "select tp.p_no, tp.p_name, "
	+" case when tl.t_code = 'T001' then '결핵' when tl.t_code = 'T002' then '장티푸스' "
	+" when tl.t_code = 'T003' then '수두' when tl.t_code = 'T004' then '홍역' else '콜레라' end as t_code, "
	+" to_char(tr.t_sdate, 'yyyy-mm-dd') as t_sdate, case when tr.t_status = '1' then '검사중' else '검사완료' end as t_status,  to_char(tr.t_ldate, 'yyyy-mm-dd') as t_ldate, "
	+" case when tr.t_result = 'X' then '미입력' when tr.t_result = 'P' then '양성' else '음성' end as t_result "
	+" from tbl_patient_202004 tp, tbl_lab_test_202004 tl, tbl_result_202004 tr "
	+" where tp.p_no = tr.p_no and tl.t_code = tr.t_code"
	+" order by p_no, t_result, t_code desc";
	PreparedStatement ps = conn.prepareStatement(sql);
	ResultSet rs = ps.executeQuery();
%>
```
### 섭틀릿을 이용해 while문 작성
```java
<table class="table" style="text-align: center; width: 750px;">
		<tr>
			<th>환자번호</th>
			<th>환자명</th>
			<th>검사명</th>
			<th>검사시작일</th>
			<th>검사상태</th>
			<th>검사완료일</th>
			<th>검사결과</th>
		</tr>
		<%while(rs.next()) { %>
		<tr>
			<td><%=rs.getString("p_no") %></td>
			<td><%=rs.getString("p_name") %></td>
			<td><%=rs.getString("t_code") %></td>
			<td><%=rs.getString("t_sdate") %></td>
			<td><%=rs.getString("t_status") %></td>
			<td><%=rs.getString("t_ldate") %></td>
			<td><%=rs.getString("t_result") %></td>
		</tr>
		<%} %>
	</table>
```
## 지역별검사건수 화면
![image](https://github.com/hwan06/health_examination_site/assets/114748934/7799efa1-6a10-4888-a265-e50cb94d8514)   
### group by를 이용해 병원코드 개수 count
```sql
<%@ page import = "java.sql.*" %>
<%@ page import = "DB.DBconnect" %>
<%
	Connection conn = DBconnect.getConnection();
	String sql = "select p_city, case when p_city = '10' then '서울' when p_city = '20' then '경기' when p_city = '30' then '강원' else '대구' end as s_city, count(p_city) as count"
			+" from tbl_patient_202004 tp, tbl_result_202004 tr "
			+" where tp.p_no = tr.p_no "
			+" group by p_city "
			+" order by p_city ";
	PreparedStatement ps = conn.prepareStatement(sql);
	ResultSet rs = ps.executeQuery();
%>
```
### 섭스틀릿 while문 작성
```java
<table class="table" style="text-align: center; width: 500px;">
			<tr>
				<th>지역코드</th>
				<th>지역명</th>
				<th>검사건수</th>
			</tr>
			
			<%while(rs.next()) {%>
			<tr>
				<td><%=rs.getString("p_city") %></td>
				<td><%=rs.getString("s_city") %></td>
				<td><%=rs.getString("count") %></td>
			</tr>
			<%} %>
		</table>
```
