# pintuitcast
一个讲解项目
<!DOCTYPE html>
<html>

	<head>
		<meta charset="UTF-8">
		<meta name="viewport" content="initial-scale=1.0,minimum-scale=1.0,maximum-scale=1.0,user-scalable=0" />
		<meta name="apple-mobile-web-app-capable" content="yes" />
		<title>拼图小游戏</title>
	</head>
	<link rel="stylesheet" href="base.css">
    <!-- <scrip|正常|t src="index.js"></script>  -->
	<body>
		<ul>

		</ul>
		<div id="stratBtn">开始滑动模式</div>
		<div id="stratBtnB">开始拼接模式</div>
		<p>预设：</p>
		<p>|弱智|</p>
		<p></p>
		<p>|天才|</p>
		<p>自定义在下面</p>
		<div id="option">
			<input type="text" id="imgSrc" placeholder="自定义图片地址" />
			<input type="text" id="row" placeholder="行" />
			<input type="text" id="col" placeholder="列" />
			<input type="submit" id="submit" value="确定" />
		</div>
	</body>
</html>

<script>
		var moving = false,
			step = 0,
			modle,
			oldChangeLiIndexs, difXe, difYe,
			li, goTime, stratTime, posC,
			dragLi, startTop, startLeft,
			btn = document.getElementById("stratBtn"),
			btnB = document.getElementById("stratBtnB"),
			imgSrc = document.getElementById("imgSrc"),
			myRow = document.getElementById("row"),
			myCol = document.getElementById("col"),
			sub = document.getElementById("submit"),
			p = document.getElementsByTagName("p"),
			ul = document.getElementsByTagName("ul")[0],
			sopportTouch = "ontouchstart" in document,
			tapClick =  "click",
			tapStart =  "mousedown",
			tapMove =  "mousemove",
			tapEnd =  "mouseup",
			option = {
				"imgSrc": "images/01.jpg",
				"row": 3,
				"col": 3,
				"width": 100
			};
		
		// sub.onclick = function() {
		// 	if (imgSrc.value) option.imgSrc = imgSrc.value;
		// 	if (myRow.value) option.row = myRow.value;
		// 	if (myCol.value) option.col = myCol.value;
		// 	if (imgSrc.value || myRow.value || myCol.value) {
		// 		init();
		// 		reset();
		// 	};
		// };

		// p[1].onclick = function() {
		// 	option.imgSrc = "images/02.jpg";
		// 	option.row = 3;
		// 	option.col = 3;
		// 	init();
		// 	reset();
		// };
		// p[2].onclick = function() {
		// 	option.imgSrc = "images/03.jpg";
		// 	option.row = 3;
		// 	option.col = 3;
		// 	init();
		// 	reset();
		// };
		// p[3].onclick = function() {
		// 	alert(1)
		// 	option.imgSrc = "images/04.jpg";
		// 	option.row = 4;
		// 	option.col = 4;
		// 	// init();
		// 	// reset();
		// };
		init();

		function init() {
			var wid = option.width,
				row = option.row,
				col = option.col,
				pos = [];
			if (goTime) {
				clearInterval(goTime)
			};
			if (stratTime) {
				clearInterval(stratTime)
			};
			step = 0;
			ul.innerHTML = "";
			btn.innerText = "开始滑动模式";
			btnB.innerText = "开始拖拽模式";
			btnB.style.backgroundColor = "salmon";
			btnB.style.backgroundColor = "salmon";
			for (var i = 0; i < row * col; i++) {
				ul.innerHTML += "<li></li>";
			}
			li = document.getElementsByTagName("li");
			ul.style.width = col * wid + "px";
			ul.style.height = row * wid + "px";
			for (var i = 0; i < option.row; i++) {
				for (var ii = 0; ii < option.col; ii++) {
					pos.push([i * wid, ii * wid]);
				};
			};
			posC = pos;
			for (var i = 0; i < li.length; i++) {
				li[i].style.backgroundImage = "url(" + option.imgSrc + ")";
				li[i].style.backgroundSize = col * wid + "px " + row * wid + "px";
				li[i].style.width = wid + "px";
				li[i].style.height = wid + "px";
				li[i].style.top = pos[i][0] + "px";
				li[i].style.left = pos[i][1] + "px";
				li[i].style.backgroundPosition = "-" + pos[i][1] + "px -" + pos[i][0] + "px";
				li[i].setAttribute("data", pos[i][0] + "," + pos[i][1]);
				li[i].indexs = i;
			};
			btn.addEventListener("click", start);
			btnB.addEventListener("click", startB);
		};

		function reset() {
			btn.style.backgroundColor = "salmon";
			btnB.style.backgroundColor = "salmon";
			for (var i = 0; i < li.length; i++) {
				li[i].removeEventListener(tapClick, clickLi);
				li[i].removeEventListener(tapStart, mouseDown);
			};
			ul.removeEventListener(tapMove, drag);
			ul.removeEventListener(tapEnd, stopDrag);
		};

		function start() {
			var _this = this;
			modle = 1;
			btnB.removeEventListener("click", startB);
			btnB.style.backgroundColor = "#808080";
			if (_this.innerText == "开始滑动模式" || _this.innerText.indexOf("过关") != -1) {
				hiddenLi = parseInt(Math.random() * li.length);
				moving = false;
				li[hiddenLi].id = "hidden";
				_this.innerText = "停止打乱";
				goTime = setInterval(function() {
					findRemovableLi();
				}, 100);
			} else if (_this.innerText == "停止打乱") {
				var times = 0;
				clearInterval(goTime);
				_this.innerText = "0s";
				stratTime = setInterval(function() {
					times++;
					_this.innerText = times + "s";
				}, 1000);
			};
			for (var i = 0; i < li.length; i++) {
				li[i].addEventListener(tapClick, clickLi);
				li[i].removeEventListener(tapStart, mouseDown);
			};
		};

		function startB() {
			modle = 2;
			btn.removeEventListener("click", start);
			btn.style.backgroundColor = "#808080";
			if (this.innerText == "开始拖拽模式") {
				disturb();
				for (var i = 0; i < li.length; i++) {
					li[i].addEventListener(tapStart, mouseDown);
					li[i].removeEventListener(tapClick, clickLi);
				};
			};
		};

		function disturb() {
			var posB = posC.slice(0);
			for (var i = 0; i < li.length; i++) {
				var posBLenght = posB.length,
					num = parseInt(Math.random() * posBLenght),
					topLeft = posB[num];
				li[i].style.top = topLeft[0] + "px";
				li[i].style.left = topLeft[1] + "px";
				posB.splice(num, 1);
			};
		};

		function mouseDown(event) {
			var e = event ? event : window.event,
				mx = sopportTouch ? e.touches[0].clientX : e.clientX,
				my = sopportTouch ? e.touches[0].clientY : e.clientY,
				ex = this.offsetLeft,
				ey = this.offsetTop;
			difXe = mx - ex;
			difYe = my - ey;
			dragLi = this;
			startTop = parseInt(this.style.top);
			startLeft = parseInt(this.style.left);
			ul.addEventListener(tapMove, drag);
			ul.addEventListener(tapEnd, stopDrag);
			ul.addEventListener("mouseleave", mouseOutUl);
		};

		function mouseOutUl() {
			dragLi.style.top = startTop + "px";
			dragLi.style.left = startLeft + "px";
			ul.removeEventListener(tapMove, drag);
		};

		function drag(event) {
			var e = event ? event : window.event,
				mx = sopportTouch ? e.touches[0].clientX : e.clientX,
				my = sopportTouch ? e.touches[0].clientY : e.clientY,
				ex = ul.offsetLeft,
				ey = ul.offsetTop,
				difX = mx - ex,
				difY = my - ey,
				top = difY - difYe,
				left = difX - difXe;
			dragLi.style.zIndex = 999;
			dragLi.style.top = top + "px";
			dragLi.style.left = left + "px";
		};

		function stopDrag(event) {
			ul.removeEventListener(tapMove, drag);
			ul.removeEventListener("mouseleave", mouseOutUl);
			dragLi.style.zIndex = 1;
			var e = event ? event : window.event,
				mx = sopportTouch ? e.changedTouches[0].clientX : e.clientX,
				my = sopportTouch ? e.changedTouches[0].clientY : e.clientY,
				ex = ul.offsetLeft,
				ey = ul.offsetTop,
				difX = mx - ex,
				difY = my - ey,
				nearestLi = findNearestLi(difY, difX),
				nearestTop = nearestLi ? parseInt(nearestLi.style.top) : startTop,
				nearestLeft = nearestLi ? parseInt(nearestLi.style.left) : startLeft;
			moveLi(dragLi, startTop, startLeft, nearestTop, nearestLeft, nearestLi);
		};
         // 拖拽
		function findNearestLi(top, left) {
			var newLi = document.getElementsByTagName("li"),
				wid = option.width,
				maxTopN = Math.floor(top / wid) * 100,
				maxLeftN = Math.floor(left / wid) * 100;
			if (maxTopN == startTop && maxLeftN == startLeft) {
				return false;
			}
			for (var i = 0; i < newLi.length; i++) {
				var liTop = parseInt(newLi[i].style.top),
					liLeft = parseInt(newLi[i].style.left);
				if (liTop == maxTopN && liLeft == maxLeftN) {
					return newLi[i];
				};
			};
		};

		function clickLi() {
			if (moving) return false;
			var _this = this,
				currentLiTop = parseInt(this.style.top),
				currentLiLeft = parseInt(this.style.left),
				blankTop = parseInt(document.getElementById("hidden").style.top),
				blankLeft = parseInt(document.getElementById("hidden").style.left);;
			if (currentLiTop == blankTop) {
				if (currentLiLeft == blankLeft - option.width || currentLiLeft == blankLeft + option.width) {
					//console.log("ok");
					moveLi(_this, currentLiTop, currentLiLeft, blankTop, blankLeft);
				};
			} else if (currentLiLeft == blankLeft) {
				if (currentLiTop == blankTop - option.width || currentLiTop == blankTop + option.width) {
					//console.log("ok");
					moveLi(_this, currentLiTop, currentLiLeft, blankTop, blankLeft);
				};
			};
		};

		function moveLi(_this, currentLiTop, currentLiLeft, blankTop, blankLeft, blankLi) {
			moving = true;
			_this.style.top = blankTop + "px";
			_this.style.left = blankLeft + "px";
			if (blankLi) {
				blankLi.style.top = currentLiTop + "px";
				blankLi.style.left = currentLiLeft + "px";
				step++;
				btnB.innerText = "已用了" + step + "步！";
			};
			if (document.getElementById("hidden")) {
				document.getElementById("hidden").style.top = currentLiTop + "px";
				document.getElementById("hidden").style.left = currentLiLeft + "px";
			};
			if (btn.innerText != "停止打乱" || btnB.innerText != "开始拖拽模式") checkComplate();
			setTimeout(function() {
				moving = false;
			}, 500);
		};
        // 开始打乱
		function findRemovableLi() {
			var blankLi = document.getElementById("hidden"),
				liList = document.getElementsByTagName("li"),
				blankLiTop = parseInt(blankLi.style.top),
				blankLiLeft = parseInt(blankLi.style.left),
				removableList = [];
			for (var i = 0; i < liList.length; i++) {
				var liTop = parseInt(liList[i].style.top),
					liLeft = parseInt(liList[i].style.left);
				if ((liTop == blankLiTop && (liLeft == blankLiLeft + option.width || liLeft == blankLiLeft - option.width)) || (liLeft == blankLiLeft && (liTop == blankLiTop + option.width || liTop == blankLiTop - option.width))) {
					removableList.push([liList[i].indexs, liTop, liLeft, i]);
				};
			};
			if (removableList.length != 0) {
				var num = parseInt(Math.random() * removableList.length),
					changeLiIndexs = removableList[num][0];
				if (changeLiIndexs == oldChangeLiIndexs) {
					findRemovableLi();
				} else {
					moveLi(liList[removableList[num][3]], removableList[num][1], removableList[num][2], blankLiTop, blankLiLeft);
					oldChangeLiIndexs = changeLiIndexs;
				};
			};
		};

		function checkComplate() {
			var newLi = document.getElementsByTagName("li"),
				complate = false;
			for (var i = 0; i < newLi.length; i++) {
				var currentLiTop = parseInt(newLi[i].style.top),
					currentLiLeft = parseInt(newLi[i].style.left),
					topLeft = currentLiTop + "," + currentLiLeft,
					data = newLi[i].attributes.data.value;
				if (data != topLeft) {
					return false
				} else {
					complate = true;
				}
			};
			if (complate) {
				if (modle == 1) {
					clearInterval(stratTime);
					setTimeout(function() {
						document.getElementById("hidden").id = "";
					}, 500);
					var txt = btn.innerText;
					btn.innerText = "过关！共用了" + txt;
					moving = true;
				} else {
					btnB.innerText = "过关！共用了" + step + "步！";
					ul.removeEventListener(tapMove, drag);
					ul.removeEventListener(tapEnd, stopDrag);
				};
				for (var i = 0; i < li.length; i++) {
					li[i].removeEventListener(tapClick, clickLi);
					li[i].removeEventListener(tapStart, mouseDown);
				};
			};
		};
		document.ontouchmove = function(e) {
			e.preventDefault();
		};
	</script>
