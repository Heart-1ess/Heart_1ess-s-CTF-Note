# Unserialize

* ### 反序列化函数赋值

protected op --&gt; \00\*\00op

class Name{

    private op --&gt; \00Name\00op

* ### \_\_wakeup 魔术方法绕过

更改序列化后变量个数使其超过真实变量个数完成绕过\_\_wakeup\(\)魔术方法



