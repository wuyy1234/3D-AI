# 3D-AI
## P&D 过河游戏智能
### 要求
*  实现状态图的自动生成
*  讲解图数据在程序中的表示方法
*  利用算法实现下一步的计算

###  实现过程
*  加入协程
> 在原来的魔鬼与牧师基础上修改，看到之前的架构发现要修改不少内容。  
> 例如原来的动作管理器没有用callback,考虑到最近又有协程这个内容可以参考  
> 于是将其用用协程和mono.StartCoroutine以及callback()实现动画效果，同时也使得代码简洁。  
> 原本的动作是一个个相对分离无法连贯的  
```
actionManager.moveBoatController(boatObject);//移动船
actionManager.toBoatController ("priest", characters, boatObject);//牧师上船
actionManager.toBoatController ("devil", characters, boatObject);//魔鬼上船
actionManager.fromBoatController("devil", characters, boatObject);//魔鬼下船
actionManager.fromBoatController("priest", characters, boatObject);//牧师下船
```
> 修改如下:  
> 例如实现一个魔鬼过河。包括三个动作：魔鬼上船，移动船，魔鬼下船  
```
mono.StartCoroutine(D());
```
> 上面是调用，下面是实现多个动作连贯  
```
public IEnumerator D(){
		actionIsFinish = false;
		actionManager.toBoatController ("devil", characters, boatObject);
		while (!actionIsFinish) {
			yield return null;
		}
		actionIsFinish = false;
		actionManager.moveBoatController(boatObject);
		while (!actionIsFinish) {
			yield return null;
		}

		actionIsFinish = false;
		actionManager.fromBoatController("devil", characters, boatObject);
		while (!actionIsFinish) {
			yield return null;
		}
	}
```
![这里写图片描述](http://imglf6.nosdn.127.net/img/Z281REhERnhNZldXM05IVmgzUWNVMnp2L09sTERIUjNTUEQvbTFQTE1wNUJZUkJDTHpDLzJnPT0.gif)  


*  加入状态图
> 因为不考虑输掉的状态，所以状态图如下：  
![这里写图片描述](http://imglf3.nosdn.127.net/img/Z281REhERnhNZldXM05IVmgzUWNVMUw1cCs2MlduMWUzcTBvQklPeU14RkFBM3d2enAwNHdnPT0.png?imageView&thumbnail=500x0&quality=96&stripmeta=0)  
> 解释：其中D表示实现一个魔鬼过河，2P2DB表示右岸有两个魔鬼牧师，船在右岸。  
> 获取各个变量如下：  
```
int[] s = new int[3]{0,0,0};
//获取船的位置
s [0] = boatObject.getBoatStatus();//1:右岸
//获取右岸牧师的数量
for (int i = 0; i < 3; i++) {//priest 1:右岸
  if(characters[i].status==1){
    s [1]++;
  }
}
//获取右岸魔鬼的数量
for (int i = 3; i < 6; i++) {//priest
  if(characters[i].status==1){
    s [2]++;
  }
}
```




