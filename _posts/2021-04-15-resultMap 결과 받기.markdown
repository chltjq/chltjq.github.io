---
layout: post
title: "Mybatis resultMap 결과 여러개 받기"
date: 2021-04-14 17:53:02 +0900
category: mybatis
---

mybatis에서 쿼리 한번에 결과를 2개의 vo로 나눠 받아야 할 일이 생겼다.

여러 방법을 찾아봤는데 vo를 수정하기도 싫고 map으로 받아 결과를 다시 넣어주기도 싫었다..

그래서 사용한 방법이 map으로 resultMap을 만들고 그 안에 collection으로 결과를 받을 vo 2개를 넣어주는 방법이였다.

```ruby
<resultMap id="PineoneBoardsVo" type="com.lguplus.test.database.dto.PineoneBoardsVo">
		<id property="boardKey"	column="board_key"	jdbcType="BIGINT"	javaType="java.lang.Long"/>
		<result property="boardTitle"	column="board_title"	jdbcType="VARCHAR"	javaType="java.lang.String"/>
		<result property="boardDesc"	column="board_desc"	jdbcType="LONGVARCHAR"	javaType="java.lang.String"/>
		<result property="regAdminKey"	column="reg_admin_key"	jdbcType="BIGINT"	javaType="java.lang.Long"/>
		<result property="modAdminKey"	column="mod_admin_key"	jdbcType="BIGINT"	javaType="java.lang.Long"/>
		<result property="regDate"	column="reg_date"	jdbcType="TIMESTAMP"	javaType="java.sql.Timestamp"/>
		<result property="modDate"	column="mod_date"	jdbcType="TIMESTAMP"	javaType="java.sql.Timestamp"/>
		<result property="cursor"	column="custom_cursor"	jdbcType="VARCHAR"	javaType="java.lang.String"/>	</resultMap>

	<resultMap id="PineoneAdminUsersVo" type="com.lguplus.test.database.dto.PineoneAdminUsersVo">
		<id property="adminKey"	column="admin_key"	jdbcType="BIGINT"	javaType="java.lang.Long"/>
		<result property="adminId"	column="admin_id"	jdbcType="VARCHAR"	javaType="java.lang.String"/>
		<result property="adminName"	column="admin_name"	jdbcType="VARCHAR"	javaType="java.lang.String"/>
		<result property="cursor"	column="custom_cursor"	jdbcType="VARCHAR"	javaType="java.lang.String"/>	</resultMap>

	<resultMap type="java.util.Map" id="BeanList">
		<collection property="board" resultMap="PineoneBoardsVo" />
		<collection property="admin" resultMap="com.lguplus.test.database.mappers.PineoneAdminUsersMapper.PineoneAdminUsersVo" />
	</resultMap>
	
	<select id="selectPineoneBoardsAndUser" parameterType="map" resultMap="BeanList">
		SELECT
		<include refid="pineoneBoardsColumns"><property name="alias" value="pb1"/><property name="prefix" value=""/></include>,
		<include refid="pineoneAdminUsersColumns"><property name="alias" value="u1"/><property name="prefix" value=""/></include>
		FROM tb_test_pineone_boards pb1 right outer join tb_test_pineone_admin_users u1 ON pb1.mod_admin_key = u1.admin_key
		WHERE pb1.board_key=#{map.boardKey} AND u1.admin_key=#{map.adminKey}
	</select>
```

sqlmap에는 이렇게 설정하고 Mapper에서는 

```
public Map<Object,Object> selectPineoneBoardsAndUser(@Param("map") Map<String, Object> searchMap);
```

이렇게 받았다. 이후 결과를 받은 Map에서 Collection property에 설정해준 값으로 key를 찾아보면 결과가 담긴 객체를 찾을 수 있었다.


조인하여 결과를 여러개 받을때 설계를 잘 해놔서 한번에 받아오면 좋겠지만 여의치 않을때 사용할 수 있을 것 같다.