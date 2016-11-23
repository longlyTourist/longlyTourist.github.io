---
layout:     post
title:      "【职场修炼】只实现功能就够了吗: 关于一个简单排列组合问题的多种思考"
subtitle:   "递归场景中JDK 1.5 CompletionService、Executor等的使用"
date:       2016-11-18 12:00:00
author:     "Silence"
header-img: "img/postbg/think.jpg"
catalog:    true
tags:
    - Java
    - Thinking in Code
---


> 本文通过一个简单的编程题，分析了多种不同Level的解题思路，目的在于启发coder新人思考的重要性：始终从用户的角度去换位思考，从“有解”到“有优秀的解”，从而达到高质交付。

> 转载请注明出处:[Silence' Blog](https://longlytourist.github.io/)。

---

# 引
前段时间看到一道Java笔试题：

> 有1、2、3、4四个数字，能组成多少个互不相同且无重复数字的三位数？都是多少？

这是一个简单的排列组合题，大家都知道这题的答案是24个；不过题面是要求用代码的方式计算出来，并输出这些数字。

别看只是一道简单的Java题，但我感到这是一个考察答题者思维深度和广度的好题，我有意将它用于公司新人的招聘上。子曾经曰过：**同样是九阴白骨爪，由周芷若和黄衫女使出来，那就是一个地下一个天上！**

解法有几种呢？我们来按武功修为从浅到深的顺序来看一看：

![image](http://ogw4950jb.bkt.clouddn.com/%E4%BA%BA%E7%89%A9/%E6%90%9E%E7%AC%91/%E6%AD%A6%E5%8A%9F%E7%A7%98%E7%B1%8D.jpg?imageMogr2/thumbnail/500x)

---

# 第一层：单刀直入

第一种思路非常地直来直去，一般刚毕业的学生很大一部分会给出这种解：

> 1. 先从4个数字中取百位数
> 2. 从剩下的3个数字中再取十位数
> 3. 剩下的2个数字依次作为个位数，并将最终产生的结果放到结果集里
> 4. 循环2~3，直到剩下的3个数字都被作为十位数用过
> 5. 循环1~4，直到所有的4个数字都被作为百位数用过
> 6. 输出最终结果


```java
// 代码略
```


有什么问题？

至少是没有考虑扩展性吧？如果不是4取3，而是5取4、6取3。。。呢？所以，我们需要把**候选数字的个数（candidateNum)**和**结果位数（numLength）**都给参数化掉吧？

想啥呢？接茬练！

![image](http://ogw4950jb.bkt.clouddn.com/%E4%BA%BA%E7%89%A9/%E6%90%9E%E7%AC%91/%E5%A6%88%E5%A6%88%E6%89%93%E8%84%B8.jpg?imageMogr2/thumbnail/500x)

---

# 第二层：隔山打牛
为了实现**候选数字个数（candidateNum)**和**结果位数（numLength）**的参数化，我们应该能想到的就是：**递归**；这就犹如隔山打牛的内家功夫：将造数计算一层层递归下去，并完成最终击倒目标的目的~ 基本逻辑思路其实和上一解法一样。

> 1. 初始化一个动态的非重候选数字列表
> 2. 开始递归造数(详细参见makeNums()递归方法，每一步都有详细的注释)
> 3. 输出最终结果

**Shut up & Show me the CODE!**

```java
public class Main {
    // 候选最小数字
    private static int FROM = 1;
    // 候选最大数字
    private static int TO = 4;
    // 需要生成的数字长度
    private static int LENGTH = 3;

    public static void main(String[] args) {
        // 初始化候选数字
        List<Integer> candidateDigits = initCandidateDigits(FROM, TO);
        if(candidateDigits == null){
            return;
        }
        // 造数
        Date startTime = new Date();
        Set<Integer> result = new TreeSet<Integer>();
        makeNums(result, "", candidateDigits, LENGTH);
        Date endTime = new Date();
        // 输出
        System.out.println("Finish! Cost(ms): " + (endTime.getTime() - startTime.getTime()) + "\n" +
                "There're " + result.size() + " diffrent nums, and they are:");
        for(int num : result){
            System.out.println(num);
        }
    }


    /**
     *  生成指定位长度非重数
     * @param result    结果集
     * @param numHead 组成头几位的数字，首次调用时应为""
     * @param candidateDigits  候选数字
     * @param remainLength 要生成的数字的位数
     * @return
     */
    public static Set<Integer> makeNums(Set<Integer> result, String numHead, List<Integer> candidateDigits, int remainLength){
        // 如果结果集为空，就创建一个
        if(result == null){
            result = new TreeSet<Integer>();
        }
        // 如果已到最后一位数（即个位数），则将剩下的数字依次拼装给“前缀数位”，并存入最终结果
        if(remainLength <= 1){
            for(int digit : candidateDigits){
                result.add(Integer.parseInt(numHead + digit));
            }
        }
        // 否则依次选取下一个位数的数字，并递归
        else{
            for(int digit : candidateDigits){
                // 新的头部数位 = 旧的头部数位 + 下一位选定的数字
                String newNumHead = numHead + digit;
                // 去除已选定的下一位数字
                List<Integer> tmpDigits = new ArrayList<Integer>();
                tmpDigits.addAll(candidateDigits);
                tmpDigits.remove(new Integer(digit));
                int newRemainLength = remainLength - 1;
                // 将剩下的候选数字和剩下的位数进行递归
                makeNums(result, newNumHead, tmpDigits, newRemainLength);
            }
        }
        return result;
    }

    /**
     * 初始化候选数字(这里只考虑1~9为合法数字）
     * @param from 候选最小数字，应小于等于to
     * @param to   候选最大数字，应大于等于from
     * @return
     */
    public static List<Integer> initCandidateDigits(int from, int to){
        List<Integer> result = new ArrayList<Integer>();
        if(from >= to || 1 > from || 9 < from || 1 > to || 9 < to){
            return null;
        }
        for(;from <= to; from++){
            result.add(from);
        }
        return result;
    }
}
```
通常实现到这一步就可以了，不过，**子又曰过：为了成就绝世武功，男人就应该对自己狠一点！**

---

# 第三层：葵花宝典

天赋异禀而又追（xin)求(li)极(bian)致(tai)的东方不败又想到：如果是大位数怎么办呢？比如是从63个64进制的数字中挑选58个数之类的？（确实够变态）而且，现在只是拼个数，万一在某些情况下，拼好了之后，要求根据这个数进行一个实时的复杂业务计算怎么办？——好的，这时候我们应该想到的是——**多线程异步**

东方不败练就了这一层武功后，能将万千银针同时急速射出，目标将瞬间毙命！**天下武功，为快不破！**用了插上多线程翅膀的递归模式，就连以快打快的独孤九剑也不是敌手！

但是，葵花宝典不是人人都能练的，在练之前有很多艰难险阻需要跨越（淫笑的那个去面壁）：

> 1. 第一个问题就是：在层层的递归嵌套面前，用什么方式来进行优雅的多线程异步？
> 2. 需要有一个机制，能知晓所有这些递归的异步线程（并不是简单的并列异步的线程）什么时候结束？基于业务的需要，可能还要获取递归的中间数据进行业务操作。。。

下面我们来一个个解（zi）决（gong）：

#### 递归排列中的多线程逻辑

> 1. 初始化一个动态的非重候选数字列表
> 2. 初始化一个异步线程池及异步服务
> 3. 用异步服务执行排列递归
> 4. 在递归方法中，需要用到递归调用的时候，利用第2步中的异步服务执行排列递归
> 5. 在**合适的时机**获取最终的排列数结果

但是，如何获取递归结果？什么才是**“合适的时机”**？

这个问题可以用JDK1.5的java.util.concurrent包解决。

#### 利用JDK1.5并发工具包实现异步机制的原理

java.util.concurrent包是1.5新增的并发工具包。具体逻辑是：

> 1. 将要异步的逻辑（本例中即排列递归的逻辑）封装在一个实现了Callable接口的类的call()方法中，使其可以被第2步中的Executor托管起来异步执行；
> 2. 依赖java.util.concurrent.Executor的实现类初始化一个java.util.concurrent.CompletionService接口的实现类；其中前者负责将托管的业务逻辑按照某种线程机制（如线程池）异步地执行；后者负责将业务逻辑的执行和执行的结果分离开来，并提供一个访问执行结果的能力；
> 3. 利用completionService的submit()方法进行执行异步逻辑
> 4. 利用completionService.take()**阻塞式**得获取执行结果；得知是否异步操作已完成；

（该包的其他具体用法请参考Java API）

太抽象？好吧，**假想一下这样一个场景**：

你和你老婆/老公很恩爱，结果没控制住，生了一打baby（**业务逻辑**）。孩子长大后要上学吧？于是你给他们每个人都办了入学证（**实现Callable接口**），使得他们能够同时入学（**被托管异步执行**）。

你带着他们来到学校（**CompletionService**）找到了校长（**Executor**),说：我孩子这么多，都要上学，你看咋整？

校长（**Executor**)说：放心交给我吧！我会安排N个老师（**多个线程**）、按照我们定制的教学大纲、在我们的多个多功能教室（**线程使用的细节、调度等**）进行系统化教学的！当然了，以上我所说的一切，**你都不需要关心！**你要做的就是：孩子想上学的时候，你可以**随时送到学校来**（**completionService.submit()**）,然后孩子上完课了（**业务逻辑完成**）你**来学校门口跟看门老大爷说一声：“我要接孩子放学”**（**completionService.take()**）就是了！当然，接了孩子回去后你可以问问他们今天都教了啥呀（**completionService.take().get()**）？


#### 如何跟踪递归的异步多线程执行情况

不过这里有一个坑，是校长没告诉你的：学校是不知道孩子几点下课的！如果你到了学校门口跟看门老大爷说：下课了吧？俺想接俺娃回家~（**completionService.take()**）

也许这时看门老大爷会甩给你一句：我也没看到你家娃出来，肯定是还没下课呢（**业务逻辑执行中**）！等着吧！（**阻塞**）

这时，你也只能干等着。。。

**解决这种尴尬的办法有两个：**

1 . 根据经验预估一个孩子上课的时长，然后到点了去接（**completionService.poll(long timeout, TimeUnit unit)**）；这种方法的优点就是：够简单！缺点则是：不够稳定！： 到了规定的时间点仍没接到你要怎么办？先回去，然后再等这么长时间再来接？这太过浪费家长的时间！还有些家长则更简单粗暴：这熊孩子！等这么久都没下课！这娃俺不要了！（知道为什么现在人贩子这么好就业了吧？）

---

2 . 相比起来，方法二则显得更为靠谱：因为要送N个娃去上学，那我就在某个地方记住我一共送了多少个娃进学校，然后接的时候我就死等，直到接满了我要接的个数，我就回家！

```java
for (int i = 0; i < n; ++i) {
    // 接娃，注意：如果没接到会在这里阻塞
    Future<Child> oneOfMyChildren = completionService.take();
    // 可能的业务处理：把娃的作业本拿出来替他写作业（可怜天下父母心）。。。
    HomeWork homeWork = oneOfMyChildren.get().getHomeWork();
    doHomeWork(homeWork);
}
```

---

3 . 可是问题又来了：因为你是在很多随意的时间点送了N个孩子去上学，甚至有些孩子是你委托保姆送的，所以导致你根本不记得今天一共送了几个？

有人说：那我在每送一个孩子上学（**completionService.submit()**）的时候就累计一下数目（**counter++**)，然后我去接的时候每接一个孩子（**completionService.take()**）就递减一下数目（**counter- -**)，接到counter为0，就说明都接完了，这不就行了？答案是：不行！

因为**接、送孩子的时机也许是不固定的**（你也许没法控制保姆接送的时机），所以多个孩子的接和送是可能发生交织的，这就导致当你的计数（**counter**)为0的时候，也有可能只是把之前送进学校的几个孩子接走了，这时候还有几个小孩正在保姆的带领下去学校呢！但这时候你武断地认为小孩都接完了就回去睡觉了（退出程序片段），那这几个娃放学后没人接，他们的家庭作业谁来做？！你还有没有点做家长的责任心？！

这时候，你要么依然采用方法1中的办法设超时时限；但另一个可行的办法是：**提前计算好今天一共要送多少小孩上学，并设置好counter**，然后管他是谁送的、在什么时候送的；只要在接的时候每接一个就递减counter，直到counter为0。**这种办法很适用于递归场景**

在本例的递归案例中，按照第二层武功代码中的递归逻辑，**递归逻辑的异步执行次数**是可以计算出来的。对于一个**P(m,n)**的递归计算（即从m个非重数字中挑选n个数字组成n位数），它在本例中的递归逻辑执行次数（包含首次调用）是：

```
Total_Counter of P(m,n)
= p(m,1) + p(m,1)*p(m-1,1)... + p(m,1)*p(m-1,1)...*p(m-n+2),1) + 1           
= m + m*(m-1) + ... + m*(m-1)*...*(m-n+2) + 1【第一次调用】
```

这是一个简单的排列组合题，不理解为什么是这个计算公式的童鞋去温习一下高中数学吧。。。

---

#### Shut up & Show me the CODE!



###### **1. 主函数**

```java
import java.util.*;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.Future;

/**
 * Created by Silence on 2016-11-18.
 */
public class Main {

    // 候选最小数字
    private static int FROM = 1;
    // 候选最大数字
    private static int TO = 5;
    // 需要生成的数字长度
    private static int LENGTH = 3;

    public static void main(String[] args) throws Exception {
        // 初始化候选数字
        List<Integer> candidateDigits = initCandidateDigits(FROM, TO);
        if(candidateDigits == null){
            return;
        }
        // 初始化异步线程池
        Date startTime = new Date();
        Set<Integer> result = Collections.synchronizedSet(new TreeSet<Integer>());
        ExecutorService service = Executors.newCachedThreadPool();
        // 初始化可跟踪排列递归次数的异步服务
        TracedRankExecutorCompletionService<Set<Integer>> completion =
                new TracedRankExecutorCompletionService<Set<Integer>>(service, candidateDigits.size(), LENGTH);
        // 造数
        completion.submit(
                new NumberMaker(result, "", candidateDigits, LENGTH, completion));
        // 确保所有递归活动都已结束
        while (true) {
            // 如果已获取完所有异步递归计算，则退出循环
            if(completion.getFutureNum() == 0){
                break;
            }
            // 如果仍有未使用的异步任务执行结果，则继续循环
            Future<Set<Integer>> future = completion.take();
            if (future != null && future.get() != null &&
                    future.isDone() && !future.isCancelled()) {
                continue;
            }
        }
        // 关闭异步服务
        service.shutdown();
        Date endTime = new Date();
        // 输出
        System.out.println("Finish! Cost(ms): " + (endTime.getTime() - startTime.getTime()) + "\n" +
                "There're " + result.size() + " diffrent nums, and they are:");
//        for(int num : result){
//            System.out.println(num);
//        }
    }

    /**
     * 初始化候选数字(这里只考虑1~9为合法数字）
     * @param from 候选最小数字，应小于等于to
     * @param to   候选最大数字，应大于等于from
     * @return
     */
    public static List<Integer> initCandidateDigits(int from, int to){
        List<Integer> result = new ArrayList<Integer>();
        if(from >= to || 1 > from || 9 < from || 1 > to || 9 < to){
            return null;
        }
        for(;from <= to; from++){
            result.add(from);
        }
        return result;
    }

}

```

###### **2. 可跟踪排列递归执行次数的CompletionService**

```java
import java.util.concurrent.*;

/**
 * 可跟踪排列递归执行次数的CompletionService
 * Created by Silence on 2016-11-22.
 */
public class TracedRankExecutorCompletionService<V> extends ExecutorCompletionService<V> {

    private static Integer futureNum = 1;// 递归次数

    /**
     * Constructor
     * @param executor
     * @param candidateNum  候选非重数字的个数
     * @param pickNum   需要挑选的数字个数
     * @throws Exception
     */
    public TracedRankExecutorCompletionService(Executor executor, int candidateNum, int pickNum) throws Exception {
        super(executor);
        if(candidateNum <= pickNum){
            throw new Exception("The candidate number must greater than the picked number");
        }
        // p(m,n)的递归次数 = p(m,1) + p(m,1)*p(m-1,1)... + p(m,1)*p(m-1,1)...*p(m-n+2),1) + 1
        //                = m + m*(m-1) + ... + m*(m-1)*...*(m-n+2) + 1【第一次调用】
        // 计算递归执行次数
        int factor = 1;
        for(int i = candidateNum; i > candidateNum - pickNum + 1; i--){
            factor *= i;
            futureNum += factor;
        }
    }

    @Override
    public Future<V> take() throws InterruptedException {
        Future<V> future = null;
        try {
            future = super.take();
            // 计数递减
            synchronized (futureNum){
                futureNum--;
            }
            return future;
        } catch (InterruptedException e) {
            throw e;
        }
    }

    public static Integer getFutureNum() {
        return futureNum;
    }
}
```

###### **3. 实现了Callable接口的递归逻辑**

```java
import java.util.*;
import java.util.concurrent.Callable;
import java.util.concurrent.CompletionService;

/**
 * Created by Silence on 2016-11-20.
 */
public class NumberMaker implements Callable<Set<Integer>> {

    private Set<Integer> result;// 结果集
    private String numHead;// 组成头几位的数字，首次调用时应为""
    private List<Integer> candidateDigits;// 候选数字
    private int remainLength;// 要生成的数字的位数
    private CompletionService<Set<Integer>> completion;// 异步服务

    // 初始构造
    public NumberMaker(Set<Integer> result, String numHead, List<Integer> candidateDigits, int remainLength, CompletionService<Set<Integer>> completion) {
        this.result = result;
        this.numHead = numHead;
        this.candidateDigits = candidateDigits;
        this.remainLength = remainLength;
        this.completion = completion;
    }

    /**
     * 生成指定位长度非重数
     */
    @Override
    public Set<Integer> call() throws Exception {
        // 如果结果集为空，就创建一个线程安全的set
        if (result == null) {
            result = Collections.synchronizedSet(new TreeSet<Integer>());
        }
        // 如果已到最后一位数（即个位数），则将剩下的数字依次拼装给“前缀数位”，并存入最终结果
        if (remainLength <= 1) {
            for (int digit : candidateDigits) {
                synchronized (result) {
                    result.add(Integer.parseInt(numHead + digit));
                }
                // 模拟复杂业务计算
                Thread.sleep(300);
            }
        }
        // 否则依次选取下一个位数的数字，并异步递归
        else {
            for (int digit : candidateDigits) {
                // 新的头部数位 = 旧的头部数位 + 下一位选定的数字
                String newNumHead = numHead + digit;
                // 去除已选定的下一位数字
                List<Integer> tmpDigits = new ArrayList<Integer>();
                tmpDigits.addAll(candidateDigits);
                tmpDigits.remove(new Integer(digit));
                int newRemainLength = remainLength - 1;
                // 将剩下的候选数字和剩下的位数进行异步递归
                completion.submit(
                        new NumberMaker(result, newNumHead, tmpDigits, newRemainLength, completion));
            }
        }
        return result;
    }
}

```

#### 运行结果比对：真的比第二层武功快吗？

注意到：在递归逻辑的call()方法中，我们加入了如下的代码来模拟一个“微小”的业务计算：

```java
    // 模拟复杂业务计算
    Thread.sleep(300);
```

我们在上文第二层武功方法的同样位置，加入同样的模拟代码，来比较一下：在加入了一个小耗时业务运算的情况下两者的差别有多大？以下是运行比较结果（表中P(M,N)表示从M个非重数字中挑选N个数字组成互不相同的N位数；）：


 比较 | 解二耗时（s） | 解三耗时（s） | 解三是解二耗时的（%）
---|--- | ---| ---
P(4，3) | 3.641 | 0.619 | 17%
p(5，3) | 6.013 | 0.969 | 16%
p(5，4) | 18.056 | 0.630 | 3.5%
p(8，7) | 没敢运行，因为大家<br>都能计算得出来，<br>其理论耗时应该在：<br>8\*7\*6\*5\*4\*3\*0.3 = 6048s | 3.797 | 0.063%

结果一目了然！再试想一下，我们只是插入了一个300ms的业务计算量，而且也只是10进制的10以内数字的排列组合，如果候选数再大店、业务运算时间再长点，解二的耗时就会成指数增长！此时，葵花宝典将具有碾压性的优势！


# 第四层：还有第四层？

葵花宝典已经是武学的最高境界了吗？当然不是！只要你善于思考，永远会不断有进步！子曰：**山外青山楼外楼，还有英雄在前头！**

在解三中至少还有很多地方是需要进一步完善的：

> - 如何支持从外部输入待选数字？
> - 如何考虑排重等容错？
> - 如果采用的是按照设置经验过期时间的方式来获取结果，那如何加入重试机制？
> - 结果太庞大的情况下，是否应该将结果输出到日志文件？
> - 为了更好的扩展性，是否应该将int都改为long？
> - ......

这里不再一一举例实现，留待大家思考。。。

最后总结一下：**只有不停得从用户角度换位思考，才能使人不断进步！只实现功能，是远远不够的！**

![](http://ogw4950jb.bkt.clouddn.com/%E4%BA%BA%E7%89%A9/%E6%90%9E%E7%AC%91/%E9%A9%AC%E6%A1%B6%E4%B8%8A%E7%9A%84%E6%80%9D%E8%80%83.jpg)
