# OS-Memory-Management
TJ OS Crouse Project
# 内存管理 - 动态分区分配方式模拟

| 学号 | 666666 |
| ---- | ------- |
| 姓名 | 777777  |

[TOC]

## 项目需求

假设初始态下，可用内存空间为640K，并有下列请求序列，请分别用首次适应算法和最佳适应算法进行内存块的分配和回收，并显示出每次分配和回收后的空闲分区链的情况来。

| 作业1申请130K |
| :-----------: |
| 作业2申请 60K |
| 作业3申请100k |
| 作业2释放 60K |
| 作业4申请200K |
| 作业3释放100K |
| 作业1释放130K |
| 作业5申请140K |
| 作业6申请 60K |
| 作业7申请 50K |
| 作业6释放 60K |

​		

### 项目目的

- 数据结构、分配算法
- 加深对动态分区存储管理方式及其实现过程的理解



## 开发环境

- **开发环境:** Windows 10
- **开发软件:** 

  1. **Visual Studio**  *2017*
- **开发语言:** C#



## 操作说明

- 双击`memoryManagement.exe`即可打开项目的可执行文件,打开后界面如下图所示

  <img src=".\img\界面.jpg" style="zoom: 67%;" />


- 进行模拟前首先要选择一种算法

![](H:\Learning\20-21-2\OS\07-王老师OS课程小项目\内存管理\1951459_赵子昱_内存管理\img\算法.jpg)

- 接下来点击**下一步**进行作业调度

<img src=".\img\下一步.jpg" style="zoom:67%;" />

- 右侧的模拟内存会显示每次分配和回收后的空闲分区链的情况*(不同作业的颜色不同, 以区分不同作业在内存中的位置分布情况)*

<img src=".\img\内存.jpg" style="zoom:67%;" />

- 下方的日志信息会显示作业*申请/释放*等信息

  <img src=".\img\日志.jpg" style="zoom: 67%;" />

- 点击**清空内存**会清空内存中的作业以及日志信息全部内容，可再次进行模拟

  <img src=".\img\清空内存.jpg" style="zoom:67%;" />

## 系统分析

- ### **首次适应算法**

  - 算法逻辑: 两个列表来记录内存使用情况，占用表记录内存中被占用部分的起始位置与长度，

    ​	空闲表记录内存中空闲部分的起始位置和长度(并将其按照物理位置顺序排序)

    ​	如果当前作业需要申请内存空间 => 顺序查找第一个空闲块大小大于所需空间 => 在占用表中加入该块 => 空闲表中相应块大小和位置做相应的调整

- ### **最佳适应算法**

  - 算法逻辑: 内存使用情况记录方法同上

    ​	如果当前作业需要申请内存空间 => 找出当前容量最小并且满足当前申请需求的物理块 =>  在占用表中加入该块 => 空闲表中相应块大小和位置做相应的调整

    

## 系统设计

### 类设计

1. **占用表与空闲表的表项类:** 也可理解为作业

   ```c#
   class tableItem
       {
           public int begin;   //起始地址
           public int length;  //块大小
           public int tag;     //此块存放的作业序号
   
           public tableItem(int begin,int length, int tag)
           {
               this.begin = begin;
               this.length = length;
               this.tag = tag;
           }
       }
   ```

2. **每个周期的动作类:** 每个周期要执行的操作

   ```c#
   //每个周期的动作
       class taskChange
       {
           public int no;          //进程编号
           public int operation;   //进程动作，0位申请，1为释放
   
           public taskChange(int no,int operation)
           {
               this.no = no;
               this.operation = operation;
           }
       }
   ```

### 实体设计

```c#
		List<tableItem> freeTable = new List<tableItem>(); 		//空闲表（记录内存空闲块）
        List<tableItem> assignedTable = new List<tableItem>();  //占用表（记录内存占用块）

        List<int> taskList = new List<int>();					 //任务列表（记录每个作业的大小）
        List<taskChange> taskChangeList = new List<taskChange>();//任务动作列表

        void initTaskChangeList()
            {
                taskChangeList.Clear();
                taskChangeList.Add(new taskChange(0, 0));
                taskChangeList.Add(new taskChange(1, 0));
                taskChangeList.Add(new taskChange(2, 0));
                taskChangeList.Add(new taskChange(1, 1));
                taskChangeList.Add(new taskChange(3, 0));
                taskChangeList.Add(new taskChange(2, 1));
                taskChangeList.Add(new taskChange(0, 1));
                taskChangeList.Add(new taskChange(4, 0));
                taskChangeList.Add(new taskChange(5, 0));
                taskChangeList.Add(new taskChange(6, 0));
                taskChangeList.Add(new taskChange(5, 1));
            }
```

## 系统实现（两个主要算法）

#### 首次适应算法

- 如果当前操作为释放某个作业

  ```c#
  if (taskChangeList[0].operation == 1)  //有作业释放
  {
      foreach (tableItem item in assignedTable)
      {
          if (item.tag == taskChangeList[0].no)
          {
              int index = dataGridView1.Rows.Add();
              dataGridView1.Rows[index].Cells[0].Value = "作业" + (taskChangeList[0].no + 1).ToString() + "释放" + taskList[taskChangeList[0].no].ToString() + "K内存空间";
              dataGridView1.Rows[index].Cells[1].Value = item.begin.ToString();
              dataGridView1.Rows[index].Cells[2].Value = colorName[taskChangeList[0].no];
              deleteTask(item);
              freeTable.Add(new tableItem(item.begin, item.length, -1));
              merge();  //整理空闲空间
              break;
          }
      }
      taskChangeList.RemoveAt(0);
  }
  ```

- 如果当前操作为某个作业申请空间

  ```c#
  else    //有作业申请空闲空间
  {
      int length = taskList[taskChangeList[0].no];
      int assignTag = 0;
      foreach (tableItem item in freeTable)
      {
          if (item.length >= length)  //找到第一个足够大的空闲空间
          {
              int index = dataGridView1.Rows.Add();
              dataGridView1.Rows[index].Cells[0].Value = "作业" + (taskChangeList[0].no + 1).ToString() + "申请" + taskList[taskChangeList[0].no].ToString() + "K内存空间";
              dataGridView1.Rows[index].Cells[1].Value = item.begin.ToString();
              dataGridView1.Rows[index].Cells[2].Value = colorName[taskChangeList[0].no];
              for (int i = 0; i < length / 10; i++)
              {
                  memoryBoxList[i+ item.begin / 10].BackColor = taskColor[taskChangeList[0].no];
              }
              assignedTable.Add(new tableItem(item.begin, length, taskChangeList[0].no));
              //更新空闲表
              if (item.length == length)
              {
                  freeTable.Remove(item);
              }
              else
              {
                  item.begin += length;
                  item.length -= length;
              }
              assignTag = 1;
              taskChangeList.RemoveAt(0);
              break;
          }
      }
      if (assignTag==0)
      {
          //添加消息
      }
  }
  ```

  

#### 最佳适应算法

- 如果当前操作为释放某个作业

  ```c#
  if (taskChangeList[0].operation == 1)  //有作业释放
  {
      foreach (tableItem item in assignedTable)
      {
          if (item.tag == taskChangeList[0].no)
          {
              int index = dataGridView1.Rows.Add();
              dataGridView1.Rows[index].Cells[0].Value = "作业" + (taskChangeList[0].no + 1).ToString() + "释放" + taskList[taskChangeList[0].no].ToString() + "K内存空间";
              dataGridView1.Rows[index].Cells[1].Value = item.begin.ToString();
              dataGridView1.Rows[index].Cells[2].Value = colorName[taskChangeList[0].no];
              deleteTask(item);
              freeTable.Add(new tableItem(item.begin, item.length, -1));
              merge();  //整理空闲空间
              break;
          }
      }
      taskChangeList.RemoveAt(0);
  }
  ```

- 如果当前操作为某个作业申请空间

  ```c#
  else    //有作业申请空间
  {
      int length = taskList[taskChangeList[0].no];
      int assignTag = 0;
      int minlen = 10000;
      foreach (tableItem item in freeTable)
      {
          if (item.length >= length&& item.length<minlen)
          {
              minlen = item.length;  //记录最小的足够大的空闲空间大小
          }
      }
  
      foreach (tableItem item in freeTable)
      {
          if (item.length == minlen)  //找到最小的足够大的空闲空间
          {
              int index = dataGridView1.Rows.Add();
              dataGridView1.Rows[index].Cells[0].Value = "作业" + (taskChangeList[0].no + 1).ToString() + "申请" + taskList[taskChangeList[0].no].ToString() + "K内存空间";
              dataGridView1.Rows[index].Cells[1].Value = item.begin.ToString();
              dataGridView1.Rows[index].Cells[2].Value = colorName[taskChangeList[0].no];
              for (int i = 0; i < length / 10; i++)
              {
                  memoryBoxList[i + item.begin / 10].BackColor = taskColor[taskChangeList[0].no];
              }
              assignedTable.Add(new tableItem(item.begin, length, taskChangeList[0].no));
              //更新空闲表
              if (item.length == length)
              {
                  freeTable.Remove(item);
              }
              else
              {
                  item.begin += length;
                  item.length -= length;
              }
              assignTag = 1;
              taskChangeList.RemoveAt(0);
              break;
          }
      }
      if (assignTag == 0)
      {
          //添加消息
      }
  }
  ```
