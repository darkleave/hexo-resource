---
title: jooq基本操作介绍
date: 2018-01-09 00:07:42
tags: [jooq]
---

**jooq基本操作介绍**

<!--more-->

## 1.1jooq select查询##

select方法接受一个SelectField查询字段集合,org.jooq.Field可以通过
生成的任意jooq.tables.*(eg:AudienceItem extends TableImpl<AudienceItemRecord>)的Fields方法获取该Table的所有Field字段或者通过该Table的公共成员变量获取部分字段(eg: AUDIENCE_ITEM.ID).

select方法返回一个**SelectSelectStep**<Record> 查询步骤对象,一般通过该对象调用from方法完成接下来的表拼接.eg:.from(AUDIENCE_OBJECT).

from方法接受一个TableLike<?>对象，该对象可以是表也可以是视图,最后返回一个**SelectJoinStep**<R>对象.

SelectJoinStep对象继承自**SelectWhereStep**对象,通过该对象我们既可以直接后接where方法条件，也可以通过SelectJoinStep对象的join,innerjoin等方法继续关联表进行查询.

SelectJoinStep对象调用join等方法后(eg:.leftJoin(AUDIENCE_ITEM))返回SelectJoinPartitionByStep<R>对象,该对象继承自**SelectOnStep**对象.

**SelectOnStep**对象的主要方法是on,通过on方法来完成join之后的条件拼装(eg:.on(AUDIENCE_OBJECT.ITEM_ID.eq(AUDIENCE_ITEM.ID)),调用完成之后返回SelectOnConditionStep<R>对象.

通过**SelectOnConditionStep**<R>对象,可以继续接join,where或者直接调用fetch方法结束sql拼接.

fetch方法返回一个泛型结果集Result<R>,一般通过该结果集的into方法直接转化得到实体列表,具体实例如下:

    DSLContext dsl = DSL.using(configuration);
    dsl.select(Fields.start().add(AUDIENCE_ITEM).end())
                    .from(AUDIENCE_OBJECT).leftJoin(AUDIENCE_ITEM)
                    .on(AUDIENCE_OBJECT.ITEM_ID.eq(AUDIENCE_ITEM.ID).and(AUDIENCE_OBJECT.BUSINESS_TYPE.eq(businessType)))
                    .where(AUDIENCE_OBJECT.BUSINESS_ID.eq(id).and(AUDIENCE_ITEM.ID.isNotNull())).fetch()
                    .into(AudienceItem.class)


另:on方法和where方法都是接受一个Condition可变数组为入参,Condition条件对象可由TableImpl对象的TableField字段调用and,eq等条件方法得到.

## 1.2 使用DSLContext进行新增,修改,删除##

    //新增
    DSL.using(conf).insertInto(AUDIENCE_OBJECT,AUDIENCE_OBJECT.ID).values("1").execute();
    //更新    	
    DSL.using(conf).update(AUDIENCE_OBJECT).set(AUDIENCE_OBJECT.BUSINESS_ID, AUDIENCE_OBJECT.BUSINESS_ID.add(1)).execute();
    //删除    	
    DSL.using(conf).delete(AUDIENCE_OBJECT).where(AUDIENCE_OBJECT.BUSINESS_ID.eq("1")).execute();
    	


## 2.1 使用UpdatableRecord完成新增,修改,删除##

jooq的新增，修改，删除方法都可以通过**UpdatableRecord**对象完成,示例如下:

    DSLContext dsl = DSL.using(conf);
    	
    UpdatableRecord r = (UpdatableRecord) dsl.newRecord(AUDIENCE_OBJECT, objects.get(0));
        	//只有不为空的才进行更新
        	int size = r.size();
    		for (int i = 0; i < size; ++i) {
    			if (r.getValue(i) != null || r.field(i).getDataType().nullable())
    				continue;
    			r.changed(i, false);//标记该字段不更新
    		}
        	r.update();
        	
        	r.insert();
        	
        	r.delete();



## 2.2 批量方法##

新增,修改,删除批量方法都是接受一个UpdatableRecord对象集合来进行批量新增或批量修改.

    //批量添加
    DSL.using(conf).batchInsert(rs).execute();
    
    //批量修改
    DSL.using(conf).batchUpdate(rs).execute();
    
    //批量删除
    DSL.using(conf).batchDelete(rs).execute();
    
    