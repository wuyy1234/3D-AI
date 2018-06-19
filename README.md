# 3D-AI
## P&D 过河游戏智能
### 要求
*  实现状态图的自动生成
*  讲解图数据在程序中的表示方法
*  利用算法实现下一步的计算

> 最终效果（如果gif卡顿请直接点击gif）  
![这里写图片描述](http://imglf5.nosdn.127.net/img/Z281REhERnhNZlVWTno5bk96U3FxUmQ0YnlPb09vU3RKMUp2ZjNpOEhWNC9uY2twa1RIVGR3PT0.gif)  
> 打乱初始情况（如果gif卡顿请直接点击gif）  
![这里写图片描述](http://imglf4.nosdn.127.net/img/Z281REhERnhNZlVWTno5bk96U3FxWnNXeXVRd2V5VHZxTkVNU0xidEpyWGhXTjRUSVdtcEd3PT0.gif)  

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
> 根据上面获取的当前状态，依据下面的判断决定下一个最优路线  
```
if( (s[0]==1&&s[1]==3&&s[2]==0)||
	(s[0]==-1&&s[1]==3&&s[2]==2)||
	(s[0]==-1&&s[1]==3&&s[2]==3)||
	(s[0]==1&&s[1]==0&&s[2]==1)||
	(s[0]==-1&&s[1]==0&&s[2]==1)||
	(s[0]==-1&&s[1]==3&&s[2]==0)||
	(s[0]==-1&&s[1]==0&&s[2]==2)
	){
	mono.StartCoroutine(D());
}
else if((s[0]==-1&&s[1]==2&&s[2]==2)
	){
	mono.StartCoroutine(P());
}
else if((s[0]==-1&&s[1]==3&&s[2]==1)||
	(s[0]==1&&s[1]==0&&s[2]==2)||
	(s[0]==1&&s[1]==0&&s[2]==3)||
	(s[0]==1&&s[1]==3&&s[2]==2)){
	mono.StartCoroutine(DD());

}
else if((s[0]==1&&s[1]==3&&s[2]==1)||
	(s[0]==1&&s[1]==2&&s[2]==2)){
	mono.StartCoroutine(PP());
}
else if((s[0]==1&&s[1]==3&&s[2]==3)||
	(s[0]==1&&s[1]==1&&s[2]==1)||
	(s[0]==-1&&s[1]==1&&s[2]==1)){
	mono.StartCoroutine(PD());
}
```
> 下面实现各个动作集的代码(只列出PD，其余的类似)：
```
public IEnumerator PD(){
	actionIsFinish = false;
	actionManager.toBoatController ("priest", characters, boatObject);
	while (!actionIsFinish) {
		yield return null;
	}
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
	actionIsFinish = false;
	actionManager.fromBoatController("priest", characters, boatObject);
	while (!actionIsFinish) {
		yield return null;
	}
}
```


