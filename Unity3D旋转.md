Unity3D 旋转函数
=====================

Transform.eulerAngles 欧拉角
----------------------------
**var eulerAngles : Vector3**   
欧拉角可以表示物体的旋转角度。
    
    // 绕x轴旋转10度，绕y轴旋转10度，绕z轴旋转10度
    transform.eulerAngles = Vector3(10, 10, 10);

+ 仅使用者这个变量读取和设置角度到绝对值。不要递增它们，当超过角度360度，它将失败。使用Transform.Rotate替代。  
+ 不要分别设置欧拉角其中一个轴（例如：eulerAngles.x = 10; ），因为这将导致偏移和不希望的旋转。当设置它们一个新的值时，要同时设置全部。


Quaternion.Euler 欧拉角
-----------------------
**static function Euler (x : float, y : float, z : float) : Quaternion**  
返回一个旋转角度，绕z轴旋转z度，绕x轴旋转x度，绕y轴旋转y度。
    
    //绕y轴旋转30度
    var rotation = Quaternion.Euler(0, 30, 0);
    
**static function Euler (euler : Vector3) : Quaternion**  
返回一个旋转角度，绕z轴旋转z度，绕x轴旋转x度，绕y轴旋转y度。
    
    //绕y轴旋转30度
    var rotation = Quaternion.Euler(Vector3(0, 30, 0));

Quaternion.Slerp 球形插值
------------------------
**static function Slerp (from : Quaternion, to : Quaternion, t : float) : Quaternion**    
用类似于球面曲线的速度t，将角度从from平滑的转到to。
    
    var from : Transform;
    var to : Transform;
    var speed = 0.1;
    function Update () {
        // 使用速度speed平滑的将物体的角度从from转变到to
	    transform.rotation = Quaternion.Slerp (from.rotation, to.rotation, Time.time * speed);
    }



Transform.rotation 旋转角度
-----------------------------
**Transform.rotation**：代表了物体在世界坐标系中的旋转角度，并且使用**四元数（Quaternion）**来存储。   
使用Transform.Rotate来旋转一个物体，使用Transform.eulerAngles设置作为欧拉角的旋转角度。





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
