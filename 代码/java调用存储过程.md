```java
// 级联删除岗位信息
public String delPostBase(final String postId, final String uuapname) {
    return getJdbcTemplate().execute(new CallableStatementCreator() {
        @Override
        public CallableStatement createCallableStatement(Connection con) throws SQLException {
            CallableStatement call = con.prepareCall("call talent.etl_talent_review"
                    + ".delete_post_cascade(?,?,?)");
            call.setString(1, uuapname);
            call.setString(2, postId);
            call.registerOutParameter(3, Types.VARCHAR);
            return call;
        }
    }, new CallableStatementCallback<String>() {
        @Override
        public String doInCallableStatement(CallableStatement cs) throws SQLException {
            cs.execute();
            return cs.getString(3);
        }
    });
}
```



