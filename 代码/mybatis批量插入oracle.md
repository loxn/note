```xml
INSERT INTO T_OPTIMUS_CL_CULOGIN_REWARD (UUID,CREATETIME,STATUS,PHONE)
<foreach item="item" index="index" collection="list" separator="union all">
	(select #{item.uuid},#{item.createTime},#{item.status},#{item.phone} from DUAL)
</foreach>
```

