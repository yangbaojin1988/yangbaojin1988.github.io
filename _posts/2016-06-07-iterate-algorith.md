---
layout: post
title:  "改进的前序遍历树算法实现无限极分类"
date:   2016-06-07 13:01:35 +0800
categories: sunnyday update
---
大家通常都是使用[递归实现无限极分类](http://www.phpddt.com/php/generateTree.html){:target="_blank"}，都知道递归效率很低，下面介绍一种改进的前序遍历树算法，不适用递归实现无限极分类，在大数据量实现树状层级结构的时候效率更高：

### 原理实现：
按树状显示数据如下， 从根节点开始（“Food”），然后他的左边写上1。然后按照树的顺序（从上到下）给“Fruit”的左边写上2。这样，你沿着树的边界是“遍历”，然后同时在每个节点的左边和右边写上数字。最后，我们回到了根节点“Food”在右边写上18。下面是标上了数字的树，同时把遍历的顺序用箭头标出来了。如下图：

![Alt text](http://www.phpddt.com/usr/uploads/2014/05/3032923764.jpg)

### 不难发现这些左右值很有规律：
- 一个节点的左值和右值中间的值节点都是改节点的后续节点，如在“Friut” 2-11中间的都是其后续节点。如果你要查找祖先节点列表，很容易发现，祖先节点左值小于该节点左值，右值大于该节点右值
- 如果相邻的两条记录的右值第一条的右值比第二条的大那么就是他的父类，注意这里是相邻记录。

### 实现过程：
- 导入下面sql语句

```sql
CREATE TABLE IF NOT EXISTS `category` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `title` varchar(50) NOT NULL,
  `lft` int(11) NOT NULL,
  `rgt` int(11) NOT NULL,
  `order` int(11) NOT NULL COMMENT '排序',
  `create_time` int(11) NOT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB  DEFAULT CHARSET=utf8 AUTO_INCREMENT=12 ;
 
INSERT INTO `category` (`id`, `title`, `lft`, `rgt`, `order`, `create_time`) VALUES
(1, '顶级栏目', 1, 20, 1, 1261964806),
(2, '编辑后的分类', 16, 19, 50, 1264586212),
(4, '公司产品', 10, 15, 50, 1264586249),
(5, '荣誉资质', 8, 9, 50, 1264586270),
(6, '资料下载', 6, 7, 50, 1264586295),
(7, '人才招聘', 4, 5, 50, 1264586314),
(8, '留言板', 2, 3, 50, 1264586884),
(9, '总裁', 17, 18, 50, 1267771951),
(10, '新的分类的子分类', 11, 14, 0, 1400044841),
(11, 'PHP点点通-http://www.phpddt.com', 12, 13, 0, 1400044901);
```

- 实现增删改查代码(注：这是PHP Codeigniter框架简单实现过程)

```php
<?php
/**
 * 纯属测试
 * 
 * @author Mckee
 * @link http://www.phpddt.com
 */
class Category extends CI_Controller {
    
    public function __construct()
    {
        parent::__construct();
        
        $this->load->database();
    }
    
    
    public function view()
    {
        $lists = $this->db->order_by('lft', 'asc')->get('category')->result_array();
        
        //相邻的两条记录的右值第一条的右值比第二条的大那么就是他的父类
        //我们用一个数组来存储上一条记录的右值，再把它和本条记录的右值比较，如果前者比后者小，说明不是父子关系，就用array_pop弹出数组，否则就保留
        
        //两个循环而已，没有递归
        $parent = array();
        $arr_list = array();
        foreach($lists as $item){
 
            if(count($parent)){
                while (count($parent) -1 > 0 && $parent[count($parent) -1]['rgt'] < $item['rgt']){
                   array_pop($parent);
                }   
            }
 
            $item['depath'] = count($parent);
            $parent[]  = $item;
            $arr_list[]= $item;
        }
        
        //显示树状结构
        foreach($arr_list as $a)
        {
            echo str_repeat('--', $a['depath']) . $a['title'] . '<br />';
        }
        
    }
    
    
    /**
     * 
     * 插入操作很简单找到其父节点，之后把左值和右值大于父节点左值的节点的左右值加上2，之后再插入本节点，左右值分别为父节点左值加一和加二
     */
    public function add()
    {
        //获取到父级分类的id
        $parent_id = 10;
        $parent_category = $this->db->where('id', $parent_id)->get('category')->row_array();
        
        #1.左值和右值大于父节点左值的节点的左右值加上2
        $this->db->set('lft', 'lft + 2', FALSE)->where(array('lft >' => $parent_category['lft']))->update('category');
        $this->db->set('rgt', 'rgt + 2', FALSE)->where(array('rgt >' => $parent_category['lft']))->update('category');
        
        #2.插入新的节点
        $this->db->insert('category', array(
            'title' => '新的分类的子分类',
            'lft' => $parent_category['lft'] + 1,
            'rgt' => $parent_category['lft'] + 2,
            'order' => 0,
            'create_time' => time()
        ));
        
        echo 'add success';
    }
    
    /**
     * 删除
     * 
     * #1.得到删除的节点，将右值减去左值然后加1，得到值$width = $rgt - $lft + 1;
     * #2.删除左右值之间的所有节点
     * #3.修改条件为大于本节点右值的所有节点，操作为把他们的左右值都减去$width
     */
    public function delete()
    {
        //通过分类id获取分类
        $id = 3;
        $category = $this->db->where('id', $id)->get('category')->row_array();
        
        //计算$width
        $width = $category['rgt'] - $category['lft'] + 1;
        
        #1.删除该条分类
        $this->db->delete('category', array('id' => $id));
        
        #2.删除左右值之间的所有分类
        $this->db->delete('category', array('lft >' => $category['lft'], 'lft <' => $category['rgt']));
        
        #3.修改其它节点的值
        $this->db->set('lft', "lft - {$width}", FALSE)->where(array('lft >' => $category['rgt']))->update('category');
        $this->db->set('rgt', "rgt - {$width}", FALSE)->where(array('rgt >' => $category['rgt']))->update('category');
        
        echo 'delete success';
        
    }
    
    
    //编辑，
    public function edit()
    {
        //不用说了， 直接通过id编辑
        $id = 2;
        
        $this->db->update('category', array(
            'title' => '编辑后的分类'
        ), array(
            'id' => $id
        ));
        
        echo 'edit success';
    }
 
}
```