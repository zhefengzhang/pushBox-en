# Using Cocos Creator to realize the Game of pushing boxes quickly

## *preface*

Before you start building our game, let’s download the tutorial from [GitHub](https://github.com/caizj-cn/pushBoxTest).You can download
the [completed](https://github.com/Jno1995/MouseHit) version as well, but try to build your game with us first.

If you have trouble with our lesson, please study the following documentation:

- Prepare the development environment
  - [Cocos Creator 2.0.8](http://cocos2d-x.org/filedown/CocosCreator_v2.0.8_win)
  - [Visual Studio Code](https://code.visualstudio.com/Download)
- [Document](https://docs.cocos.com/creator/manual/en/)
  - [Action system](https://docs.cocos.com/creator/manual/en/scripting/actions.html)
  - [Action list](https://docs.cocos.com/creator/manual/en/scripting/action-list.html)
  - [User data storage](https://docs.cocos.com/creator/manual/en/advanced-topics/data-storage.html?h=%E6%95%B0)

------

### 1. Functional module division:

The game has designed a total of four functional modules,They are:

- Start the game (**menuLayer**)
- Level selection (**levelLayer**)
- Game play view (**gameLayer**)
- Game settlement (**gameOverLayer**)

##### 1.1 Add main script for the game

Right-click the `Script` folder, create a new `JavaScript` script, and rename it "gameLayer". The script is then added to the Canvas node of the gameLayer scene. After adding the necessary property inspector parameters to the script, its code details are as follows:

```
//gameLayer.js code
cc.Class({
extends: cc.Component,

properties: {
	bg : cc.Node,
	menuLayer : cc.Node,
	levelLayer : cc.Node,
	gameLayer : cc.Node,
	gameControlLayer : cc.Node,
	gameOverLayer : cc.Node,
	//menuLayer view element
	startBtn : cc.Node,
	titleImg : cc.Node,
	iconImg : cc.Node,
	loadingTxt : cc.Node,

	//levelLayer view element
	levelScroll : cc.Node,
	levelContent : cc.Node,

	//gameLayer view element
	levelTxt : cc.Node,
	curNum : cc.Node,
	bestNum : cc.Node,

	//spriteAltlas
	itemImgAtlas: cc.SpriteAtlas,
	levelImgAtlas: cc.SpriteAtlas,

	levelItemPrefab : cc.Prefab,
	}
}
```

After the code is added, save the code and see the effect in the editor:

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190822142643358.png)

Each property is exposed as a box in the property inspector panel of the component.The box additional information identified by the yellow box represents the type of the property.Only resources, nodes, and components that are consistent with this type can be placed in the box, and their important information will be recorded in the `.fire` file of the scene.After the binding is complete, the effect is as follows:

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190822142847600.png)

#### 2. Design menuLayer module

All the resources required for the game are already in the `assets` directory of the [teaching project](https://github.com/caizj-cn/pushBoxTest). You can use these resources directly to start designing the game directly. At present, the game is still lack of a game start interface, we began to design the first interface for the scene: `menuLayer`.

##### 2.1 Layout for the menuLayer interface

Double-click the *gameLayer* scene in the *Scene* folder to enter the scene editor interface.In the teaching project, we have retained the UI layout that has been designed, and you can also use your imagination to adjust the scene design.The resource path for this interface is: "assets/resources/Texture/img/".The following is the layout page for reference (For the teaching of scene making, please refer to this document:[Scene production workflow](https://docs.cocos.com/creator/manual/en/content-workflow/))：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190822142916318.png)

The menuLayer node is displayed by default at the beginning of the game.The display and hiding of each level can be controlled by code to realize the switching of different modules. Interface design ideas: after the player enters the game, click on the start game button, trigger the button click on the callback function, function execution after loading and display the game level selection interface. The callback function code for the start game button is as follows:

```
//gameLayer.js code
//callback of the start game button
startBtnCallBack(event, customEventData){
    if(this.curLayer == 1){
        return;
    }
    
	this.curLayer = 1;
	
	this.playSound(sound.BUTTON);       

	this.menuLayer.runAction(cc.sequence(
    	cc.fadeOut(0.1),
    	cc.callFunc(() => {
        	this.startBtn.stopAllActions();
        	this.startBtn.scale = 1.0;
        	this.menuLayer.opacity = 255;
        	this.menuLayer.active = false;
    	})
	));

	this.levelLayer.active = true;
	this.levelLayer.opacity = 0;
	this.levelLayer.runAction(cc.sequence(
    	cc.delayTime(0.1), 
    	cc.fadeIn(0.1), 
    	cc.callFunc(() => {
        	this.updateLevelInfo();
    	})
	));
}
```

The button callback function is then added to the Click Events array of the `cc.Button` component. You can do this in the Cocos Creator editor, as follows:
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190822142943832.png)

And to start the game button to add a small animation, so that the node constantly zoom in and out. The main use is api in `action system`. The code is as follows:

```
//gameLayer.js code
//start button animation
menuLayerAni(){
    this.startBtn.scale = 1.0;
    this.startBtn.runAction(cc.repeatForever(cc.sequence(
        cc.scaleTo(0.6, 1.5), 
        cc.scaleTo(0.6, 1.0)
    )));
},
```

Save the code, run the game, and preview it in web, and you can see the following effects:

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190822143009148.gif)

#### 3. Design levelLayer module

The design of the level selection interface is divided into two parts:

- Level interface display
  - By reading the configuration file, loading the level prefabricated resources consistent with the configuration data, displaying all levels
- Update level information
  - Update each level information according to the game

#### 3.1 Level interface display

All levels in the game will be placed in the [ScrollView](https://docs.cocos.com/creator/manual/en/components/scrollview.html?h=scroll) component node.First, create the checkpoint node ”levelItem“ in the Content node of the ScrollView.

![在这里插入图片描述](https://img-blog.csdnimg.cn/2019082214330034.png)

Then drag the node into the Prefab folder, make a prefabricated resource levelItem, and then delete the checkpoint node under the content node.

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190822143243155.png)

During the loading process of the level interface, the height of the `content` node of the ScrollView component is recalculated after reading the level configuration file, and then loading the level that conforms to the configuration file record. The following is the reference code for the level interface to load and display the level node:

```
//gameLayer.js code
//create a checkpoint interface sub-node
createLavelItem (){
    //select level callback
    let callfunc = level => {            
        this.selectLevelCallBack(level);
    };
	//load and display levelItem
    for(let i = 0; i < this.allLevelCount; i++){
        let node = cc.instantiate(this.levelItemPrefab);
        node.parent = this.levelScroll;
        let levelItem = node.getComponent("levelItem");
        levelItem.levelFunc(callfunc);
        this.tabLevel.push(levelItem);
    }
    //set container height
    this.levelContent.height = Math.ceil(this.allLevelCount / 5) * 135 + 20;
},
```

#### 3.2 Update level information

Add a levelItem script component to the levelItem preform.

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190822143222861.png)

The levelItem script component is responsible for updating information. It mainly controls whether it is clickable, the number of customs stars, the level of the level, and the click to enter. The levelItem script component updates the UI code as follows:

```
//levelItem.js code
/**
 * @description: Show the number of stars
 * @param {boolean} isOpen   //whether or not to turn it on
 * @param {starCount} Number of stars
 * @param {cc.SpriteAtlas} levelImgAtlas   //texture map
 * @param {number} level 
 * @return: 
 */
showStar(isOpen, starCount, levelImgAtlas, level){
    this.itemBg.attr({"_level_" : level});
    if(isOpen){
        this.itemBg.getComponent(cc.Sprite).spriteFrame = levelImgAtlas.getSpriteFrame("pass_bg");
        this.starImg.active = true;
        this.starImg.getComponent(cc.Sprite).spriteFrame = levelImgAtlas.getSpriteFrame("point" + starCount);
        this.levelTxt.opacity = 255;
        this.itemBg.getComponent(cc.Button).interactable = true;
    }
    else{
        this.itemBg.getComponent(cc.Sprite).spriteFrame = levelImgAtlas.getSpriteFrame("lock");
        this.starImg.active = false;
        this.levelTxt.opacity = 125;
        this.itemBg.getComponent(cc.Button).interactable = false;
    }
    this.levelTxt.getComponent(cc.Label).string = level;
},
```

You can store the player's customs clearance information through `cc.sys.localStorage` API. The status of customs clearance can be divided into three states: cleared, just opened and unopened. The specific implementation is as follows:

```
// gameLayer.js code
// refresh the information on the level
updateLevelInfo(){
    let finishLevel = parseInt(cc.sys.localStorage.getItem("finishLevel") || 0);  //get finish level record
    for(let i = 1; i <= this.allLevelCount; i++){
        // finish level
        if(i <= finishLevel){
            let data = parseInt(cc.sys.localStorage.getItem("levelStar" + i) || 0);
            this.tabLevel[i - 1].showStar(true, data, this.levelImgAtlas, i);
        }
        // new round
        else if(i == (finishLevel + 1)){
            this.tabLevel[i - 1].showStar(true, 0, this.levelImgAtlas, i);
        }
        // unopened level
        else{  
            this.tabLevel[i - 1].showStar(false, 0, this.levelImgAtlas, i);
        }
    }
},
```

Save the code, run the preview, click the start game button to see the level interface loaded and displayed, the following is a preview:

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190822143203967.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM0MTIxNzg0,size_16,color_FFFFFF,t_70)



#### 4. Design gameLayer module

The design of the gameLayer module is also divided into two parts:

- Load and display the interface
- Game operation judgment

#### 4.1 Load and display the interface

Use levelConfig.json to configure each level of information in the game.![在这里插入图片描述](https://img-blog.csdnimg.cn/20190822143144566.png)

Each level game part consists of multiple rows and columns of squares. Each level information contains `content`, `allRow`, `allCol`, `heroRow`, `heroCol`, `allBox` attributes, `allRow` and `allCol` record the total number of rows and columns, `heroRow` and `heroCol` record the location of heroes, and `allBox` records the total number of boxes. 'content` is the core, record the attributes of each box, and display different objects, such as walls, floors, objects, and boxes, according to different attributes. You can add any level by modifying the configuration. 
Read all the data at the level and display different objects according to the properties of each location. The main core code is as follows:

```
//gameLayer.js code
//create a level
createLevelLayer(level){
    this.gameControlLayer.removeAllChildren();
    this.setLevel();
    this.setCurNum();
    this.setBestNum();
    let levelContent = this.allLevelConfig[level].content;
    this.allRow = this.allLevelConfig[level].allRow;
    this.allCol = this.allLevelConfig[level].allCol;
    this.heroRow = this.allLevelConfig[level].heroRow;
    this.heroCol = this.allLevelConfig[level].heroCol;
  
    // calculate block size
    this.boxW = this.allWidth / this.allCol;
    this.boxH = this.boxW;
  
    // calculation of starting coordinates
    let sPosX = -(this.allWidth / 2) + (this.boxW / 2);
    let sPosY = (this.allWidth / 2) - (this.boxW / 2);
  
    // calculate the offset of coordinates, operation rules (wide spread, set high coordinates)
    let offset = 0;
    if(this.allRow > this.allCol){
        offset = ((this.allRow - this.allCol) * this.boxH) / 2;
    }
    else{
        offset = ((this.allRow - this.allCol) * this.boxH) / 2;
    }
    this.landArrays = [];   //map container
    this.palace = [];       //initialize map data
    for(let i = 0; i < this.allRow; i++){
        this.landArrays[i] = [];  
        this.palace[i] = [];
    }
  
    for(let i = 0; i < this.allRow; i++){    //row
        for(let j = 0; j < this.allCol; j++){     //col
            let x = sPosX + (this.boxW * j);
            let y = sPosY - (this.boxH * i) + offset;
            let node = this.createBoxItem(i, j, levelContent[i * this.allCol + j], cc.v2(x, y));
            this.landArrays[i][j] = node;
            node.width = this.boxW;
            node.height = this.boxH;
        }
    }
  
    // show character
    this.setLandFrame(this.heroRow, this.heroCol, boxType.HERO);
 },
```



```
//gameLayer.js code
//create elements based on type
createBoxItem(row, col, type, pos){
    let node = new cc.Node();
    let sprite = node.addComponent(cc.Sprite);
    let button = node.addComponent(cc.Button);
    sprite.spriteFrame = this.itemImgAtlas.getSpriteFrame("p" + type);
    node.parent = this.gameControlLayer;
    node.position = pos;
    if(type == boxType.WALL){  //Metope, named wall_row_col
        node.name = "wall_" + row + "_" + col;
        node.attr({"_type_" : type});
    }
    else if(type == boxType.NONE){  //Blank area, named none_row_col
        node.name = "none_" + row + "_" + col;
        node.attr({"_type_" : type});
    }
    else{  //game interface, named land_row_col
        node.name = "land_" + row + "_" + col;
        node.attr({"_type_" : type});
        node.attr({"_row_" : row});
        node.attr({"_col_" : col});
        button.interactable = true;
        button.target = node;
        button.node.on('click', this.clickCallBack, this);
        if(type == boxType.ENDBOX){  //boxes at the target point, add 1 directly to the number of boxes completed
            this.finishBoxCount += 1;
        }
    }
    this.palace[row][col] = type;
    return node;
},
```

All elements of the game, placed in the gameControlLayer node in the following figure:

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190822143116940.png)

After the start of the game, the effects displayed are as follows (the first level, other levels are similar)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190822143058281.png)

#### 4.2 Game operation judgment

After the route is calculated, the player moves. If the player clicks on the box area, first detect whether there are obstacles in front of the box. If not, push the box. By switching the pictures of the map and modifying the location type to achieve the effect of pushing the box.Click on the map location to get the optimal path, and the character runs to the specified point. The implementation code is as follows:

```
//gameLayer.js code
//click on the map element
clickCallBack : function(event, customEventData){
  let target = event.target;
  //minimum path length
  this.minPath = this.allCol * this.allRow + 1;
  //optimal route
  this.bestMap = [];
  //end position
  this.end = {};
  this.end.row  = target._row_;
  this.end.col = target._col_;
  
  //starting point position
  this.start = {};
  this.start.row = this.heroRow;
  this.start.col = this.heroCol;
  
  //determine the type of end point
  let endType = this.palace[this.end.row][this.end.col];
  if((endType == boxType.LAND) || (endType == boxType.BODY)){  //It is an open space or a target point, and the trajectory is calculated directly.
      this.getPath(this.start, 0, []);
      if(this.minPath <= this.allCol * this.allRow){
          this.bestMap.unshift(this.start);
          this.runHero();
      }else{
          console.log("Unable to find path to reach");
      }
  }
  else if((endType == boxType.BOX) || (endType == boxType.ENDBOX)){ //It's a box. Determine if it can be pushed.
      //calculate the distance between the box and the character
      let lr = this.end.row - this.start.row;
      let lc = this.end.col - this.start.col;
      if((Math.abs(lr) + Math.abs(lc)) == 1){  //The box is in the upper, lower, left and right directions of the characters
          //calculate if there are any obstacles in the propulsion azimuth
          let nextr = this.end.row + lr;
          let nextc = this.end.col + lc;
          let t = this.palace[nextr][nextc];
          if(t && (t != boxType.WALL) && (t != boxType.BOX) && (t != boxType.ENDBOX)){  //there are no obstacles, no walls, push the box.
              this.playSound(sound.PUSHBOX);
              //character position restoration
              this.setLandFrame(this.start.row, this.start.col, this.palace[this.start.row][this.start.col]);
  
              //box location type
              let bt = this.palace[this.end.row][this.end.col];
              if(bt == boxType.ENDBOX){      //the type of box with a target object, restored to the target point
                  this.palace[this.end.row][this.end.col] = boxType.BODY;
                  this.finishBoxCount -= 1;
              }
              else{
                  this.palace[this.end.row][this.end.col] = boxType.LAND;
              }
              //the location of the box becomes a figure, but the type is saved as an empty space or target point
              this.setLandFrame(this.end.row, this.end.col, boxType.HERO);
  
              //the position in front of the box becomes a box
              let nt = this.palace[nextr][nextc];
              if(nt == boxType.BODY){  //there is a target point, set the box type to have a target box
                  this.palace[nextr][nextc] = boxType.ENDBOX;
                  this.finishBoxCount += 1;
              }
              else {
                  this.palace[nextr][nextc] = boxType.BOX;
              }
              this.setLandFrame(nextr, nextc, this.palace[nextr][nextc]);
  
              this.curStepNum += 1;
              //refresh step
              this.setCurNum();
              
              //refresh character position
              this.heroRow = this.end.row;
              this.heroCol = this.end.col;
  
              this.checkGameOver();
          }
          else{
              this.playSound(sound.WRONG);
          }
      }
      else{   //target point error
          this.playSound(sound.WRONG);
      }
  }
},

```

```
//gamePlayer.js code
//get the optimal path algorithm, curPos records the current coordinates, step records the number of steps
getPath : function(curPos, step, result){
    //determine whether or not to reach the end point
    if((curPos.row == this.end.row) && (curPos.col == this.end.col)){
        if(step < this.minPath){
            this.bestMap = [];
            for(let i = 0; i < result.length; i++){
                this.bestMap.push(result[i]);
            }
            this.minPath = step; //if the current number of arrival steps is smaller than the minimum value, modify the minimum value
            result = [];
        }
    }
    //recursion
    for(let i = (curPos.row - 1); i <= (curPos.row + 1); i++){
        for(let j = (curPos.col - 1); j <= (curPos.col + 1); j++){
            //jump over the line
            if((i < 0) || (i >= this.allRow) || (j < 0) || (j >= this.allCol)){
                continue;
            }
            if((i != curPos.row) && (j != curPos.col)){//Ignore bevel
                continue;
            }
            else if(this.palace[i][j] && ((this.palace[i][j] == boxType.LAND) || (this.palace[i][j] == boxType.BODY))){
                let tmp = this.palace[i][j];
                this.palace[i][j] = boxType.WALL;  //Marked as unwalkable

                //save alignment
                let r = {};
                r.row = i;
                r.col = j;
                result.push(r);

                this.getPath(r, step + 1, result);
                this.palace[i][j] = tmp;  //Try to end, unmark
                result.pop();
            }
        }
    }
}

```

#### 5. design gameOverLayer module

After the end of the game, according to the success of pushing to the number of boxes, to determine whether the game is successful, after the success of the game, update the level information. 
The code details are as follows:

```
//gameLayer.js code
//game end detection
checkGameOver(){
    let count = this.allLevelConfig[this.curLevel].allBox;
    // all pushed to the specified position.
    if(this.finishBoxCount == count){   
        this.gameOverLayer.active = true;
        this.gameOverLayer.opacity = 1; 
        this.gameOverLayer.runAction(cc.sequence(
            cc.delayTime(0.5), 
            cc.fadeIn(0.1)
        ));

        // number of levels completed by refresh
        let finishLevel = parseInt(cc.sys.localStorage.getItem("finishLevel") || 0);
        if(this.curLevel > finishLevel){
            cc.sys.localStorage.setItem("finishLevel", this.curLevel);
        }

        // refresh star level
        cc.sys.localStorage.setItem("levelStar" + this.curLevel, 3);

        // refresh the most Uber number
        let best = parseInt(cc.sys.localStorage.getItem("levelBest" + this.curLevel) || 0);
        if((this.curStepNum < best) || (best == 0)){
            cc.sys.localStorage.setItem("levelBest" + this.curLevel, this.curStepNum);
        }
        this.playSound(sound.GAMEWIN);
        this.clearGameData();
    }
},

```

Interface layout reference:

[外链图片转存失败(img-Gt0l6Cbi-1566455180327)(C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1566406219190.png)]

### 6. Introduction of the author

The author of this tutorial is from [CocosCreator Chinese Forum](https://forum.cocos.com/) user: **caizj.cn**, Go to its [forum home page](https://forum.cocos.com/users/caizj.cn/activity) and [github home page](https://github.com/caizj-cn) to learn more about his contribution to CocosCreator game development.

------

## Thanks to the author for his excellent articles to help developers learn CocosCreator game development.
