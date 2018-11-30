###
  TreeMap默认是升序的，如果我们需要改变排序方式
https://www.cnblogs.com/liujinhong/p/6113183.html
package com.atoomu.xingwa;

import java.util.Comparator;
import java.util.Iterator;
import java.util.Map;
import java.util.Set;
import java.util.TreeMap;


public class App 
{
    public static void main( String[] args )
    {
        Map<String, String> map = new TreeMap<String, String>(new Comparator<String>() {
        	public int compare(String obj1, String obj2) {
        		return obj2.compareTo(obj1);
        		}
        	});
        map.put("c", "ccccc");
        map.put("a", "aaaaa");
        map.put("b", "bbbbb");
        map.put("d", "ddddd");
        
        Set<String> keySet = map.keySet();
        Iterator<String> iter = keySet.iterator();
        while (iter.hasNext()) {
            String key = iter.next();
            System.out.println(key + ":" + map.get(key));
        }
    }
}

###
