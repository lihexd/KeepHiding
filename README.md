# Tizen-App-KeepHiding
### 概述
KeepHiding基于web project开发，本篇主要介绍KeepHiding的核心算法。
### 算法介绍
主要使用了Html与Javascript技术。通过使用两个计时器setInterval(ballMove, ball)和setInterval(ballCreate, add)控制礼物炸弹的移动和产生，礼物炸弹每2000ms产生随机速度（有上限和下限）、随机位置（屏幕上30%）和产生随机类型。

	function ballMove(){
		for(var i=0;i<ballArray.length;i++){
			ballArray[i].clearBall();
		}
		for(var i=0;i<ballArray.length;i++){
			ballArray[i].drawBall();
		}	
		for(var i = 0;i<ballArray.length;i++){
			var ballX=ballArray[i].getLocationX();
	        var ballY=ballArray[i].getLocationY();
	        var personLocation = person.getLocation();
	        var distence = Math.sqrt(Math.pow((height-personRadius-ballY),2)+Math.pow((personLocation-ballX),2));
	        //console.log(distence);
	        if(distence<=ballRadius + personRadius){
				var colorIndex = 0;
				for(;colors.length;colorIndex ++){
					if(ballArray[i].getColor() == colors[colorIndex])
						break;
				}
				switch(colorIndex)
				{
					case 0: 
						gameOver = true;
						timer.stop();
						//alert("die");
						$("#myModal").modal(); 
						clearInterval(ballInterval);
						clearInterval(addInterval);
						timer.stop();
						break;
					case 1:
						ballArray[i].clearBall();
						ballArray.splice(i,1);
						clearInterval(ballInterval);
						clearInterval(addInterval);
						setTimeout("resetMotion(orgBall,orgAdd)",5000);
						break;
					case 2:
						ballArray[i].clearBall();
						ballArray.splice(i,1);
						resetMotion(orgBall*5,orgAdd*5);
						setTimeout("resetMotion(orgBall,orgAdd)",5000);
						break;
					case 3:
						ballArray[i].clearBall();
						ballArray.splice(i,1);
						person.setOpposite(true);
						setTimeout("person.setOpposite(false);",5000);
						break;
					case 4:
						ballArray[i].clearBall();
						ballArray.splice(i,1);
						person.setFreeze(true);
						setTimeout("person.setFreeze(false);",5000);
						break;

				}
	        }
		}
		fixHide()
	}

	function fixHide()
	{
		for(var i = 0;i<ballArray.length;i++){
			ballArray[i].clearBall();
		}
		for(var i = 0;i<ballArray.length;i++){
			ballArray[i].redrawBall();
		}
		person.clearPerson();
		person.drawPerson();
	}

	function reset(){
		gameOver = false;
		pause = false;
		cxt.clearRect(0,0,width,height);
		ballArray.splice(0,ballArray.length);
		clearInterval(ballInterval);
		clearInterval(addInterval);
		var ball=new Ball(ballRadius,maxSpeed);
		ball.init();
		ballArray.push(ball);
		person=new Person(personRadius,width/2,personSpeed);
		person.drawPerson();
		ballInterval = window.setInterval(ballMove, orgBall);
		addInterval = window.setInterval(ballCreate, orgAdd);
		if(timer != null)
			timer.destroy();
		timer = $('#someTimer').TimeCircles({
			time : {
				Minutes: {
					show: true,
					text: "min",
					color: "#BFB"
				},
				Seconds: {
					show: true,
					text: "sec",
					color: "#F99"
				}

			},
		refresh_interval: 0.1,
		count_past_zero: true,
		circle_bg_color: "#eee",
		fg_width: 0.05,
		bg_width: 1
		});
	}
	
	
	function ballCreate(){
		var ball=new Ball(ballRadius,maxSpeed);
		ball.init();
		if(ballArray.length<40){
			ballArray.push(ball);
		}
	  }
	  
	  //创建一个小球的类
	  function Ball(radius,maxSpeed){
	  	var ballColor = "";
	  	var locationX = 0;
	  	var locationY = 0;
	  	var speedX = 0;
	  	var speedY = 0;
		this.clearBall=function(){
			cxt.clearRect(locationX-radius,locationY-radius,2*radius,2*radius);
		}
		this.init = function(){
			ballColor = this.randomColor(0,(colors.length-1)+3);
			locationX = this.randomLocation(2*radius,width-2*radius);
			locationY = this.randomLocation(2*radius,height/4);
			speedX = this.randomSpeed(1,maxSpeed);
			speedY = this.randomSpeed(1,maxSpeed);
		}
		this.redrawBall = function(){
			this.circleRender(locationX,locationY,radius,ballColor);
		}
		this.drawBall=function(){
			this.calNextLocation();
			this.circleRender(locationX,locationY,radius,ballColor);
		}
		this.calNextLocation=function(){
			this.calNextSpeed();
			locationX=locationX+speedX;
			locationY=locationY+speedY;
		}
		this.calNextSpeed=function(){
			if(locationX<radius||locationX>width-radius){
			speedX=-speedX;
			}
			if(locationY<radius||locationY>height-radius){
				speedY=-speedY;
			}
		}
		this.getSpeedX = function(){
			return speedX;
		}
		this.getSpeedY = function(){
			return speedY;
		}
		this.setSpeedX = function(speed){
			speedX = speed;
		}
		this.setSpeedY = function(speed){
			speedY = speed;
		}
		this.getLocationX=function(){
              return locationX;
		}
		this.getLocationY=function(){
             return locationY;
		}
		this.randomLocation = function(min,max){
	   		return Math.floor(min+Math.random()*(max-min));
		}
		this.randomSpeed = function(min,max){
			return Math.floor(min+Math.random()*(max-min));
		}
		this.getColor = function(){
			return ballColor;
		}
		this.randomColor = function(Min,Max)
		{   
			var Range = Max - Min;   
			var Rand = Math.random();   
			var colorIndex = Min+Math.round(Rand * Range);
			if(colorIndex >= colors.length)
				return colors[0];
			else
				return colors[colorIndex];
		}    
		this.circleRender = function(x,y,r,color){
			var img=new Image();
			img.src=color; 
			cxt.drawImage(img,x-r,y-r,r*2,r*2);
		}
	}


	function Person(personRadius,personLocation,personSpeed){
		this.opposite = false;
		this.freeze = false;
		this.setOpposite = function(opposite){
			this.opposite = opposite;
		}
		this.setFreeze = function(freeze){
			this.freeze = freeze;
		}
		this.wantGoLeft = function(){
			if(this.freeze == true)
				return;
			if(this.opposite == true)
				this.goRight();
			else
				this.goLeft();
		}
		this.wantGoRight = function(){
			if(this.freeze == true)
				return;
			if(this.opposite == true)
				this.goLeft();
			else
				this.goRight();
		}
		this.goLeft=function(){
			if(personLocation>personRadius){
			cxt.clearRect(personLocation-personRadius,height-2*personRadius,2*personRadius,2*personRadius);
			personLocation=personLocation-personRadius;
			fixHide();
	    	}
		}
		this.goRight=function(){
			if(personLocation<width-personRadius){
			cxt.clearRect(personLocation-personRadius,height-2*personRadius,2*personRadius,2*personRadius);
			personLocation=personLocation+personRadius;
			fixHide();
	    	}
		}
		this.clearPerson=function(){
			cxt.clearRect(personLocation-personRadius,height-2*personRadius,2*personRadius,2*personRadius);
		}
		this.drawPerson=function(){
			var img=new Image();
			img.src="./images/hero.png"; 
			cxt.drawImage(img,personLocation-personRadius,height-personRadius*2,personRadius*2,personRadius*2);
		}
		this.getLocation=function(){
              return personLocation;
		}

	}
	
$(function(){  
	$("#start").click(function(){
		$("#layer").css("display","none");
		var now = new Date;
		/*min = now.getMinutes();
    	sec = now.getSeconds();*/
		reset();

	});
	$("#information").click(function(){
		$("#title").css("display","none");
		$("#start").css("display","none");
		$("#information").css("display","none");
		// var innerHtml=;
		$(".guide").css("display","block");
		$("#return").css("display","block");
		// alert("hahaha");
	});
	$("#return").click(function(){
		 //window.location.reload();//刷新当前页面.
		 $("#title").css("display","block");
		$("#start").css("display","block");
		$("#information").css("display","block");
		// var innerHtml=;
		$(".guide").css("display","none");
		$("#return").css("display","none");
	});

	$("#restart").click(function(){
		reset();

	});


	$("#someTimer").click(function(){
		$("#layer").css("display","block");
		reset();
		clearInterval(ballInterval);
		clearInterval(addInterval);
		timer.stop();
		 $("#title").css("display","block");
		$("#start").css("display","block");
		$("#information").css("display","block");
		// var innerHtml=;
		$(".guide").css("display","none");
		$("#return").css("display","none");
	});
	
	window.addEventListener('load', function () {
		FastClick.attach(document.body);
	}, false);
	
	window.onbeforeunload = exit_handler;  
    window.onunload = exit_handler;  
    function exit_handler(){  
        clearInterval(ballInterval);
		clearInterval(addInterval);
    } 

});
