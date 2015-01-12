Unity3D 旋转函数
=====================

Transform.eulerAngles 欧拉角
----------------------------
**var eulerAngles : Vector3**   
旋转作为欧拉角度。
    
    // 绕x轴旋转10度，绕y轴旋转10度，绕z轴旋转10度
    transform.eulerAngles = Vector3(10, 10, 10);

+ 仅使用者这个变量读取和设置角度到绝对值。不要递增它们，当超过角度360度，它将失败。使用Transform.Rotate替代。  
+ 不要分别设置欧拉角其中一个轴（例如：eulerAngles.x = 10; ），因为这将导致偏移和不希望的旋转。当设置它们一个新的值时，要同时设置全部。

Transform.Rotate
-----------------
**function Rotate (eulerAngles : Vector3, relativeTo : Space = Space.Self) : void**    
应用一个欧拉角的旋转角度，eulerAngles.z度围绕z轴，eulerAngles.x度围绕x轴，eulerAngles.y度围绕y轴。

    //沿着x轴每秒1度慢慢的旋转物体
	transform.Rotate(Vector3.right * Time.deltaTime);

    //相对于世界坐标沿着y轴每秒1度慢慢的旋转物体
	transform.Rotate(Vector3.up * Time.deltaTime, Space.World);

**function Rotate (xAngle : float, yAngle : float, zAngle : float, relativeTo : Space = Space.Self) : void**    
应用一个旋转角度，zAngle度围绕z轴，xAngle度围绕x轴，yAngle度围绕y轴。

	//围绕x轴每秒1度，慢慢的旋转物体
	transform.Rotate(Time.deltaTime, 0, 0);

	//相对于世界坐标，围绕y轴每秒1度，慢慢的旋转物体
	transform.Rotate(0, Time.deltaTime, 0, Space.World);

**function Rotate (axis : Vector3, angle : float, relativeTo : Space = Space.Self) : void**  
按照angle度围绕axis轴旋转变换。
    
	//围绕x轴每秒1度，慢慢的旋转物体
	transform.Rotate(Vector3.right, Time.deltaTime);

	//相对于世界坐标，围绕y轴每秒1度，慢慢的旋转物体
	transform.Rotate(Vector3.up, Time.deltaTime, Space.World);
