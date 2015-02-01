Quaternion 四元数
=======================
**Quaternion (float x, float y, float z, float w)**  
使用x,y,z,w来代表一个四元数，可以通过数组下标的形式来访问。  

Unity中使用Quaternion（四元数）表示所有旋转。  

下面列出一些常用的函数和方法：

Quaternion.eulerAngles （欧拉角）
-----------------------
**Vector3 eulerAngles**  
返回表示旋转的欧拉角。    
    
	public Quaternion rotation = Quaternion.identity;
	public void Awake() {
        // 绕x旋转0度，y旋转30度，z旋转0度
		rotation.eulerAngles = new Vector3(0, 30, 0);
		print(rotation.eulerAngles.y);
	}

Quaternion.identity（单位旋转）
-------------------
相当于无旋转（这个物体完全对齐于世界或者父轴）
    
    transform.rotation = Quaternion.identity;

Quaternion.Angle (角)
-------------------------
**float Angle (Quaternion a, Quaternion b)**  
返回a和b两者之间的角度。
    
    public Transform target;
	void Update() {
        //计算transform和target之间的旋转角度
		float angle = Quaternion.Angle(transform.rotation, target.rotation);
	}

Quaternion.AngleAxis（角轴）
----------------------------
**Quaternion AngleAxis (float angle, Vector3 axis)**  
绕axis轴旋转angle，创建一个旋转。  
    
    //设置变换的旋转，绕y轴旋转30度
    transform.rotation = Quaternion.AngleAxis(30, Vector3.up);

Quaternion.Euler（欧拉角）
-------------------
返回一个旋转角度，绕z轴旋转z度，绕x轴旋转x度，绕y轴旋转y度。  

**Quaternion Euler (float x, float y, float z)**  
    
    //绕y轴旋转30度
    Quaternion rotation = Quaternion.Euler(0, 30, 0);
    
**Quaternion Euler (Vector3 euler)**  
    
    //绕y轴旋转30度
    public Quaternion rotation = Quaternion.Euler(new Vector3(0, 30, 0));

Quaternion.operator *（运算符 乘法）
--------------------
**Quaternion operator * (Quaternion lhs, Quaternion rhs)**  
旋转一个点，首先用ihs旋转（左边的旋转），再用rhs旋转（右边的旋转）。  
例一：

	public Transform extraRotation;
	public void Awake() {
		//应用extraRotation的旋转给当前旋转
        transform.rotation *= extraRotation.rotation;
	}

例二：

    // q1，q2的角度相等
    Quaternion q1 = new Quaternion();
    q1.eulerAngles = new Vector3(10, 30, 20);  

    Quaternion q2 = new Quaternion();
    Quaternion qx = Quaternion.AngleAxis(10, Vector3.right);          
    Quaternion qy = Quaternion.AngleAxis(30, Vector3.up);  
    Quaternion qz = Quaternion.AngleAxis(20, Vector3.forward);
    Quaternion q2 = qz*qy*qx;

**Vector3 operator * (Quaternion rotation, Vector3 point)**  
用rotation旋转point点
    
    public Vector3 relativeDirection = Vector3.forward;
	void Update() {
        // 旋转后点的方向
		Vector3 absoluteDirection = transform.rotation * relativeDirection;
        // 沿着新的方向移动
		transform.position += absoluteDirection * Time.deltaTime;
	}
Quaternion.Slerp（球形插值）
------------------------
**Quaternion Slerp (Quaternion from, Quaternion to, float t)**    
用类似于球面曲线的速度t，将角度从from平滑的转到to。
	
    public Transform from;
	public Transform to;
	public float speed = 0.1F;
	void Update() {
		transform.rotation = Quaternion.Slerp(from.rotation, to.rotation, Time.time * speed);
	}

Quaternion.FromToRotation（from到to旋转）
------------------------
**Quaternion FromToRotation (Vector3 fromDirection, Vector3 toDirection)**  
创建一个从fromDirection到toDirection的旋转
    
    // 将Y轴转向Z轴
    transform.rotation = Quaternion.FromToRotation(Vector3.up, transform.forward);

Quaternion.LookRotation（注视旋转）
-------------------------
**Quaternion LookRotation (Vector3 forward, Vector3 upwards)**  
创建一个旋转，沿着forward（z轴）并且头部沿着upwards（y轴）的约束注视。也就是建立一个旋转，使z轴朝向view  y轴朝向up。      

    public Transform target;
	void Update() {
        Vector3 relativePos = target.position - transform.position;
		Quaternion rotation = Quaternion.LookRotation(relativePos);
		transform.rotation = rotation;

        //上面代码等同于：
		transform.LookAt(target);
	}
