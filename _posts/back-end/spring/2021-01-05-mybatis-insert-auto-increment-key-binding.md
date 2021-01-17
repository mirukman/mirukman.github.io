---
title: MyBatis에서 insert 처리 후 auto_increment되는 key를 얻는 방법
author: mirukman
date: 2021-01-05 22:00:00 +0800
categories: [BACKEND, SPRING]
tags: [mybatis, auto_increment, key]
---

마이바티스에서 매퍼를 통해 insert를 실행하면 레코드가 생성된다. 그런데 생성된 레코드가 auto_increment 되면서 자동으로 할당되는 key가 필요할 때가 있다.

예를 들면 게시글을 등록할 때 게시글과 같이 저장되지만 서로 다른 클래스로 정의된 첨부파일 객체에 게시글의 번호를 지정해야 하는 경우다. 첨부파일은 외래키(FK)로 게시글 번호(no)를 갖는 경우다.

<br>

먼저 게시글의 VO 클래스이다.

~~~ java
@Data
public class BoardVo {
    
    private Long no;
    private String title;
    private String content;
    private String writer;
}
~~~

그리고 아래가 BoardVo의 매퍼 설정이다.

~~~ java
...
public void insertSelectKey(BoardVo boardVo);
...
~~~

~~~ xml
...

<insert id="insertSelectKey">
    <selectKey keyProperty="no" order="AFTER" resultType="Long">
        select last_insert_id()
    </selectKey>
    insert into tbl_board (title, content, writer)
    values (#{title}, #{content}, #{writer})
</insert>

...
~~~

쿼리에는 no 필드가 없다. auto_increment되는 primary key 이기 때문이다.

그러나 위와 같이 '<SelectKey>'를 설정하면 BoardVo 객체의 no 필드에 값이 저장된다. 이후부터는 해당 객체의 getter를 통해 참조를 할 수가 있게된다.

예시는 아래와 같다.

~~~ java
@Transactional
public void register(BoardVo boardVo) {
	boardMapper.insertSelectKey(boardVo); //boardVo를 저장하는 쿼리가 실행된 후 boardVo 객체의 no 필드에 값이 set 된다.
	
	//이후 첨부파일 객체의 no 필드를 지정한 후 저장할 수 있게 된다.
	if (boardVo.getAttachList() != null && boardVo.getAttachList().size() > 0) {
		boardVo.getAttachList().forEach(attach -> {
			attach.setNo(boardVo.getNo());
			attachMapper.insert(attach);
		});
	}
}
~~~

<br>

DB 종류마다 '<selectKey>'의 설정 방법이 달라 아래에 정리했다.

~~~ xml
//oracle
<selectKey keyProperty="key" resultType="Long" order="AFTER">
	SELECT idx_test_seq.currval FROM dual
</selectKey>
	
//MS-SQL
<selectKey keyProperty="key" resultType="Long" order="AFTER">
	SELECT IDENT_CURRENT('seq_test')
</selectKey>

- MariaDB(Mysql)
<selectKey keyProperty="key" resultType="Long" order="AFTER">
	SELECT LAST_INSERT_ID()
</selectKey>
~~~