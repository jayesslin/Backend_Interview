#leetcode_medium

- 146. LRU Cache


_146_LRUCache cache = new _146_LRUCache( 2 /* capacity */ );
>
   cache.put(1, 1);
>
   cache.put(2, 2);
>   
   cache.get(1);       // returns 1
>  
   cache.put(3, 3);    // evicts key 2
>   
   cache.get(2);       // returns -1 (not found)
>   
   cache.put(4, 4);    // evicts key 1
>
   cache.get(1);       // returns -1 (not found)
>  
   cache.get(3);       // returns 3
>  
   cache.get(4);       // returns 4
   
	put的值， 在头， 尾巴失效

 - 注意双端链表， 前后两个节点的before，next都需要操作
 
数据结构：
 ![](http://ww2.sinaimg.cn/large/006tNc79ly1g4am3pe7g8j30um0redh6.jpg)

```java

import java.util.HashMap;



public class _146_LRUCache {
    class listnode {
        listnode before;
        listnode next;
        int value;
        int key;
    }
    HashMap cache = new HashMap<Integer, listnode>();
    int count;
    int capacity;
    listnode head = new listnode();
    listnode tail = new listnode();

    public _146_LRUCache(int capacity) {
        this.capacity = capacity;
        this.count = 0;
        head.before=null;
        head.next= tail;
        tail.before = head;
        tail.next=null;
    }

    public int get(int key) {
        if(cache.get(key)!=null){
            listnode upkey =  (listnode)cache.get(key);

            if(upkey.before==null){
                return upkey.value;
            }
            //把需要升序的key前后连接
            upkey.before.next = upkey.next;
            upkey.next.before = upkey.before;
            //把头的第一个连接到升key
            listnode downkey = head.next;
            head.next=upkey;
            upkey.next =downkey;
            downkey.before = upkey;
            System.out.println("键值为"+key+" ， 升");
            return upkey.value;
        }
        return -1;
    }

    public void put(int key, int value) {
        //生成一个点
        listnode tmp1 = new listnode();
        tmp1.value = value;
        tmp1.key = key;
        //map取不到值
        if(cache.get(key)==null) {
            if (count < capacity) {
                System.out.println("添加链表操作");
                tmp1.next = head.next;
                head.next.before =tmp1;
                head.next = tmp1;
                tmp1.before=head;
                count += 1;
            } else {
                System.out.println("cache已满，删除链表操作");
                listnode before_delete = tail.before.before;
                System.out.println("最后一个节点值是： "+tail.before.value);
                //map删除
                cache.remove(tail.before.key);
                System.out.println("正在删除节点： "+tail.before.key +" 该节点值为："+tail.before.value);
                //链表删除
                before_delete.next = tail;
                tail.before = before_delete;
                //插入头部
                tmp1.next = head.next;
                head.next.before =tmp1;
                head.next = tmp1;
            }
        }else {

            //最后放入hashmap
            get(key);
        }
        cache.put(key, tmp1);
        readlist();
    }

    public void readlist(){
        listnode read = head;
        while(read.next.next!=null){
            System.out.println("key为"+read.next.key+"  值为 ： "+read.next.value);
            read = read.next;
        }
        System.out.println("当前count： "+count);
    }
    @test
    public static void main(String args[]){
        _146_LRUCache test = new _146_LRUCache(2);
        System.out.println(test.get(2));
        test.put(2,1);
        test.put(1,1);
        test.put(2,3);
        test.put(4,1);
        System.out.println(test.get(1));
        System.out.println(test.get(2));
        test.readlist();
    }
}

```

