---
layout:     post                    # 使用的布局（不需要改）
title:       Android ViewGroup-EditView使用
    # 标题 
subtitle:     EditView详解  #副标题
date:       2018-7-23             # 时间
author:     BY  wangchuanwen         # 作者
header-img: img/post-bg-2015.jpg    #这篇文章标题背景图片
catalog: true                       # 是否归档
tags:                               #标签
    - Android
    - EditView
    - ViewGroup
---

# EditView过滤空格

##前段时间我们有一个需求需要过滤空格；类似于密码不允许输入空格；查找api和资料

如下：

      public static void inputFilterSpace(final EditText edit){
      
      edit.setFilters(new InputFilter[]
        
                            {
                                            new InputFilter.LengthFilter(16),
                                               new InputFilter(){    
                                                   public CharSequence filter(CharSequence src, int start, int end, Spanned dst, 
                                                   int        dstart, int dend) {   
                     
                                                      
                                                      if(src.length()<1) {
         
         return null;
     
       }else

      {
    
    char temp [] = (src.toString()).toCharArray();
    
    char result [] = new char[temp.length]
    
    for(int i = 0,j=0; i< temp.length; i++){
    
    if(temp[i] == ' '){
                                                            
                                                            continue;
 
       }else{
                                result[j++] = temp[i];
                                      
                                      }
                                      
                    }
                    
    return String.valueOf(result).trim();
    
         }
         
             }
             
        } 
        
    });    
    
    }


### 我看了上面的代码后封装了一个方法；可以随时调用：


       public  String inputFilterSpace(String card){

          char temp [] = (card.toString()).toCharArray();
          
          char result [] = new char[temp.length];
          
          for(int i = 0,j=0; i< temp.length; i++){
          
              if(temp[i] == ' '){
              
                  continue;
                  
              }else{
              
                  result[j++] = temp[i];
                  
              }
              
          }
          
          return String.valueOf(result).trim();

      }
      

参考资料：<https://blog.csdn.net/fuuckwtu/article/details/6968766/>


