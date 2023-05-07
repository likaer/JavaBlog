timezone在spring boot应用中的坑



1. 日期格式作为web入参的时候，需要明确应用的TimeZone，否则默认为UST
2. DateFormat会基于timeZone做一层日期转换，这层设置会与Spring Boot应用的Timezone互相影响
3. 日期传递到数据库会做一层时区转换，比如应用为CST，传递到UST的数据库连接池，则数据时间会发生变化

