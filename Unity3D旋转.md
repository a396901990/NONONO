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
