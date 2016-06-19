title:
  Hibernate中1+N问题
date:
  2015/7/7 17:34:10
categories: Hibernate
tags: [Hibernate,ManyToOne,OneToMany,Java]
---
![image](http://ww4.sinaimg.cn/mw690/7c2c72d3gw1f2ojtjgebvj21kw0fsjsw.jpg)
# 1+N问题也可以叫N+1问题，什么是1+N问题呢？
如果在一个对象里关联另一个对象，并且fetch = FetchType.EAGER(ManyToOne的默认fetch为EAGER，OneToMany的默认fetch为LAZY)
``` java
@Entity
public class Mag {
        ...
	@ManyToOne
	@JoinColumn(name="topicId")
	public Topic getTopic() {
		return topic;
	}
	public void setTopic(Topic topic) {
		this.topic = topic;
	}
}
```
<!-- more -->
比如说ManyToOne（OneToMany也存在这种问题）关联，本来只需要取Many里的对象属性，可是Many里关联的对象都会单独再发一条语句取关联对象的属性
``` java
@Test
public void testLoadMag_1() {
	Session session = sf.getCurrentSession();
	session.beginTransaction();
	List<Mag> mags = (List<Mag>)session.createQuery("from Mag").list();
	for (Mag m : mags) {
		System.out.println(m.getId() + " - " + m.getName());
	}
	session.getTransaction().commit();
}
```
本来只用发一条就可以查出Many里的对象属性，可是它发了一条语句后，再发N条语句取关联对象的数据
``` sql
/* 只需要1条查询语句 */
Hibernate: 
    select
        mag0_.id as id1_0_,
        mag0_.cDate as cDate2_0_,
        mag0_.name as name3_0_,
        mag0_.topicId as topicId4_0_ 
    from
        Mag mag0_
/* 多出来的N条查询语句 */
Hibernate: 
    select
        topic0_.id as id1_1_0_,
        topic0_.name as name2_1_0_ 
    from
        Topic topic0_ 
    where
        topic0_.id=?
......
Hibernate: 
    select
        topic0_.id as id1_1_0_,
        topic0_.name as name2_1_0_ 
    from
        Topic topic0_ 
    where
        topic0_.id=?
```

# 解决办法
1. fetch = FetchType.LAZY：在合适的时候才发出语句（按需要发语句）
``` java
@Entity
public class Mag {
        ...
    /*将ManyToOne的fetch值设置为fetchType.LAZY*/
	@ManyToOne(fetch=fetchType.LAZY)
	@JoinColumn(name="topicId")
	public Topic getTopic() {
		return topic;
	}
	public void setTopic(Topic topic) {
		this.topic = topic;
	}
}
```
2. BatchSize：在One对象设置Size后，取出Many里的数据后，再发N/Size条语句取关联对象的数据，从而达到少发语句的目的
``` java
@Entity
/*使用BatchSize注解来设置One对象的Size大小*/
@BatchSize(size=10)
public class Topic {
	...	
}
```
3. Join Fetch：将Many与One做外连接，因此只要发一条语句就可以查出Many与其相关联的One对象数据，Criteria默认就是这种做法
```java
@Test
public void testLoadMag_1() {
	Session session = sf.getCurrentSession();
	session.beginTransaction();
/*使用外连接，来解决1+N问题*/
	List<Mag> mags = (List<Mag>)session.createQuery("from Mag m left join fetch m.topic t").list();
/*使用createCriteria方法，来解决1+N问题*/
List<Mag> mags = (List<Mag>)session.createCriteria(Mag.class).list();
	for (Mag m : mags) {
		System.out.println(m.getId() + " - " + m.getName());
	}
	session.getTransaction().commit();
}
```
# 如何选择解决办法
* 如果只要用Many里的对象，不用关联对象的属性，那就用方法1解决
* 如果要Many里的对象属性，也想要关联的对象属性就用方法3解决