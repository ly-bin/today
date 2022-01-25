##### 1、题目：https://leetcode-cn.com/problems/implement-rand10-using-rand7/。   中等 
##### 2、思路：
> 1、多个单独的随机事件相关联，关联的逻辑是多个单元累加，但是每个单元出现的概率要一定是一样的，如：两个独立的rand7会产生两个[1，7]的数，然后再用一个rand7产生一个随机数[1,2]的随机数，如果是1，
> 则随机使用一个rand7产生的值，如果是2，则第一个随机值+第二个随机值；

##### 3、代码
```
public int rand10() {
        int first, second;
        while ((first = rand7()) > 6);
        while ((second = rand7()) > 5);
        return (first&1) == 1 ? second : 5+second;
    }
```
