# [Animate CC] - Hướng dẫn làm game Drag and Drop (Matching)

###### tags: `AnimateCC`, `HTML5 Canvas`

Chào các bạn, hôm nay chúng ta sẽ cùng học cách làm game drag and drop trên Animate CC nhé!

Game drag and drop là một dạng game phổ biến tương tự như trò nối câu hỏi với đáp án của trẻ nhỏ vậy, chúng ta cần phải drag đáp án vào nơi mà chúng được mô tả (kết quả).

![](https://i.imgur.com/oyehHI0.png)

Có rất nhiều phiên bản của trò chơi này, chẳng hạn như ở vế đáp án ta có một loạt đáp án đúng sai, và bên ô kết quả thì có thể là bất cứ thứ gì để mô tả đáp án đó, chẳng hạn như một vài từ ngữ, một hình dáng để liên tưởng, hoặc thứ gì có thể suy ra từ đáp án đó... Tuy nhiên tất cả chúng đều có chung 1 logic, đó là drag đáp án vào đúng nơi của nó.

Bắt tay làm thôi.

Đầu tiên chúng ta cần phải có một số hình ảnh để dùng làm đáp án. Ta lựa chọn công cụ vẽ hình khối

![](https://i.imgur.com/iGdUyFy.png)

Vào library và tạo các symbol dựa trên các hình này nào. Lần lượt ta có hình vuông (square), hình bo góc (rounded), tròn (oval), đa giác (polygon) và ngôi sao (star)

![](https://i.imgur.com/WchtE3y.png)

Sau đó kéo hết chúng vào một cái symbol mới, mình đặt tên là pieces. Lưu ý là mọi người để position của pieces ở trên scene là (0,0) để trùng với gốc tọa độ (0,0) của scene nhé.

![](https://i.imgur.com/Y1g8TCz.png)

Trong symbol pieces, chúng ta đặt instance name cho từng symbol piece trong đó. Ở đây, mình đặt instance name nó giống với tên symbol luôn.

![](https://i.imgur.com/NI0ZCQH.png)

Tiếp theo, ta chuẩn bị phần đáp án. Ở đây mình sẽ dùng chính symbol hình khối kia để làm đáp án nhé.

Duplicate 5 symbol kia và kéo vào một symbol mới gọi là slots. Kéo slots vào scene và chỉnh origin về (0,0).

Nhấn edit symbol và thay fill thành màu xám, và stroke style thành nét đứt. Voila! Chúng ta đã có thành phẩm là slots kết quả.

![](https://i.imgur.com/BTJSOGV.png)

Đặt instance name cho 2 symbol này và bắt đầu code thôi. Trước tiên khai báo các biến thường sử dụng.

```
var root = this;
var pieces = root.pieces;
var slots = root.slots;
```

Đầu tiên là hàm init() khởi tạo đầu tiên. Để có thể drag chuột cũng như touch drag được thì cần "bật" tính năng này lên đã.

```
root.init = function() {
	createjs.Touch.enable(stage);
	stage.mouseMoveOutside = true;
};

init();
```

Đối với canvas, sự kiện drag chuột trên đó được hiểu như vẽ trên bản vẽ (draw trên canvas). Vì vậy canvas có các sự kiện drawStart, draw và drawEnd biểu thị cho hành động bắt đầu vẽ, đang vẽ và dừng vẽ.

Ta khai báo sự kiện drawStart ở hàm init. Và khi drawStart thì sẽ chạy tiếp hàm start thực hiện các hành động khi bắt đầu chọn phần tử.

```
root.init = function() {
	...
    root.drawStart = stage.on("drawstart", root.start, null, true);
};

root.start = function(e) {
	...
};
```

Khi bắt đầu, ta cần lưu lại vị trí của các symbol pieces để có thể khôi phục lại chúng nếu drag sai. Ta tạo một biến positions, lưu lại các vị trí này. Ngoài ra thì ta cần một function để xử lý khi "chọn" symbol để drag.

```
var positions = [];

root.start = function (e) {
	
    stage.off("drawstart", root.drawStart);
	
    pieces.children.forEach(function(child, index) {
        positions[index] = {x:child.x, y:child.y};
    });

    pieces.on("mousedown", root.mouseDownHandler);	
}

root.mouseDownHandler = function (e) {
    ...
}
```

Để handler sự kiện drag, ta cũng sẽ có xử lý tương tự 3 phần start - doing - end. Ta sẽ khai báo 3 hàm handler lần lượt là mouseDown, mouseMove và mouseUp để xử lý lần lượt cho các sự kiện startDrag - drag - endDrag.

Khi bắt đầu drag thì ta cần biết đối tượng được drag là gì. Ta lưu nó và biến pieces và đặt là target. Khi drag thì ta kiểm tra target này, và drag xong thì bỏ nó đi bằng cách lưu giá trị null.

```
root.mouseDownHandler = function(e)
{
	e.currentTarget.setChildIndex(e.target, e.currentTarget.children.length - 1);
	e.target.offsetX = (e.stageX / stage.scaleX) - e.target.x;
	e.target.offsetY = (e.stageY / stage.scaleY) - e.target.y;
	pieces.target = e.target;
	root.stageMouseMove = stage.on("stagemousemove", root.stageMouseMoveHandler);
	root.stageMouseUp = stage.on("stagemouseup", root.stageMouseUpHandler);
};

root.stageMouseMoveHandler = function(e)
{
	if (pieces.target)
	{
		pieces.target.x = (e.stageX / stage.scaleX) - pieces.target.offsetX;
		pieces.target.y = (e.stageY / stage.scaleY) - pieces.target.offsetY;
	}	
};

root.stageMouseUpHandler = function(e)
{
	stage.off("stagemousemove", root.stageMouseMove);
	stage.off("stagemouseup", root.stageMouseUp);
	
	if (pieces.target)
	{
		root.check();
		pieces.target = null;
	}	
};
```

Ở hàm mouseUp, ta có function check(). Function này sẽ dùng để kiểm tra xem tại vị trí mà ta drag target, có tồn tại kết quả đúng với nó không. Có 2 trường hợp xảy ra, nếu trùng (match) thì ta sẽ khớp vị trí của target với result, và nếu trật (miss) thì ta sẽ trả target về vị trí cũ của nó.

```
root.check = function()
{
	var spot = slots.getObjectUnderPoint(pieces.target.x, pieces.target.y);
	
	if (!spot)
	{
		root.onMiss();
		return;
	}
	
	root.slot = spot.parent;
		
	if (root.slot)
	{		
		if (pieces.target.name === root.slot.name)
		{
			root.onMatch();
			
			if (pieces.count === pieces.children.length)
				root.onWin();
		}
		else
			root.onMiss();
		
		root.slot = null;
	}
	else
		root.onMiss();
};

root.onMatch = function()
{	
	pieces.target.mouseEnabled = false;
	pieces.count++;
	createjs.Tween.get(pieces.target).to({x:root.slot.x, y:root.slot.y}, 350, createjs.Ease.backInOut);
};

root.onWin = function()
{
	winMessage.text = "YOU WIN!";
	winMessage.alpha = 0;
	winMessage.y = winMessage.originalY + 60;
	createjs.Tween.get(winMessage).to({alpha:1, y:winMessage.originalY}, 500, createjs.Ease.backInOut);
};

root.onMiss = function()
{	
	createjs.Tween.get(pieces.target).to({x:pieces.target.originalX, y:pieces.target.originalY}, 350, createjs.Ease.backInOut);
};
```

Khi tất cả target đã được đặt vào đúng result của nó, ở hàm onWin, chúng ta cho hiện đoạn text thông báo chiến thắng.

Sự kiện reload và shuffle. Chúng ta có thể thay đổi vị trí x, y của các phần tử trong mảng pieces và set nó lại mỗi khi reload trò chơi, như vậy sẽ tạo ra nhiều phương án hơn.
```
root.shuffle = function()
{	
	positions.sort(function(a, b)
	{
		return 0.5 - Math.random();
	});
	
	pieces.children.forEach(function(child, index)
	{
		child.originalX = positions[index].x;
		child.originalY = positions[index].y;		
		child.mouseEnabled = true;		
		createjs.Tween.get(child).to({x:child.originalX, y:child.originalY}, 350, createjs.Ease.backInOut);
	});
};
```