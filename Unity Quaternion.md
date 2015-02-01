Quaternion ��Ԫ��
=======================
**Quaternion (float x, float y, float z, float w)**  
ʹ��x,y,z,w������һ����Ԫ��������ͨ�������±����ʽ�����ʡ�  

Unity��ʹ��Quaternion����Ԫ������ʾ������ת��  

�����г�һЩ���õĺ����ͷ�����

Quaternion.eulerAngles ��ŷ���ǣ�
-----------------------
**Vector3 eulerAngles**  
���ر�ʾ��ת��ŷ���ǡ�    
    
	public Quaternion rotation = Quaternion.identity;
	public void Awake() {
        // ��x��ת0�ȣ�y��ת30�ȣ�z��ת0��
		rotation.eulerAngles = new Vector3(0, 30, 0);
		print(rotation.eulerAngles.y);
	}

Quaternion.identity����λ��ת��
-------------------
�൱������ת�����������ȫ������������߸��ᣩ
    
    transform.rotation = Quaternion.identity;

Quaternion.Angle (��)
-------------------------
**float Angle (Quaternion a, Quaternion b)**  
����a��b����֮��ĽǶȡ�
    
    public Transform target;
	void Update() {
        //����transform��target֮�����ת�Ƕ�
		float angle = Quaternion.Angle(transform.rotation, target.rotation);
	}

Quaternion.AngleAxis�����ᣩ
----------------------------
**Quaternion AngleAxis (float angle, Vector3 axis)**  
��axis����תangle������һ����ת��  
    
    //���ñ任����ת����y����ת30��
    transform.rotation = Quaternion.AngleAxis(30, Vector3.up);

Quaternion.Euler��ŷ���ǣ�
-------------------
����һ����ת�Ƕȣ���z����תz�ȣ���x����תx�ȣ���y����תy�ȡ�  

**Quaternion Euler (float x, float y, float z)**  
    
    //��y����ת30��
    Quaternion rotation = Quaternion.Euler(0, 30, 0);
    
**Quaternion Euler (Vector3 euler)**  
    
    //��y����ת30��
    public Quaternion rotation = Quaternion.Euler(new Vector3(0, 30, 0));

Quaternion.operator *������� �˷���
--------------------
**Quaternion operator * (Quaternion lhs, Quaternion rhs)**  
��תһ���㣬������ihs��ת����ߵ���ת��������rhs��ת���ұߵ���ת����  
��һ��

	public Transform extraRotation;
	public void Awake() {
		//Ӧ��extraRotation����ת����ǰ��ת
        transform.rotation *= extraRotation.rotation;
	}

������

    // q1��q2�ĽǶ����
    Quaternion q1 = new Quaternion();
    q1.eulerAngles = new Vector3(10, 30, 20);  

    Quaternion q2 = new Quaternion();
    Quaternion qx = Quaternion.AngleAxis(10, Vector3.right);          
    Quaternion qy = Quaternion.AngleAxis(30, Vector3.up);  
    Quaternion qz = Quaternion.AngleAxis(20, Vector3.forward);
    Quaternion q2 = qz*qy*qx;

**Vector3 operator * (Quaternion rotation, Vector3 point)**  
��rotation��תpoint��
    
    public Vector3 relativeDirection = Vector3.forward;
	void Update() {
        // ��ת���ķ���
		Vector3 absoluteDirection = transform.rotation * relativeDirection;
        // �����µķ����ƶ�
		transform.position += absoluteDirection * Time.deltaTime;
	}
Quaternion.Slerp�����β�ֵ��
------------------------
**Quaternion Slerp (Quaternion from, Quaternion to, float t)**    
���������������ߵ��ٶ�t�����Ƕȴ�fromƽ����ת��to��
	
    public Transform from;
	public Transform to;
	public float speed = 0.1F;
	void Update() {
		transform.rotation = Quaternion.Slerp(from.rotation, to.rotation, Time.time * speed);
	}

Quaternion.FromToRotation��from��to��ת��
------------------------
**Quaternion FromToRotation (Vector3 fromDirection, Vector3 toDirection)**  
����һ����fromDirection��toDirection����ת
    
    // ��Y��ת��Z��
    transform.rotation = Quaternion.FromToRotation(Vector3.up, transform.forward);

Quaternion.LookRotation��ע����ת��
-------------------------
**Quaternion LookRotation (Vector3 forward, Vector3 upwards)**  
����һ����ת������forward��z�ᣩ����ͷ������upwards��y�ᣩ��Լ��ע�ӡ�Ҳ���ǽ���һ����ת��ʹz�ᳯ��view  y�ᳯ��up��      

    public Transform target;
	void Update() {
        Vector3 relativePos = target.position - transform.position;
		Quaternion rotation = Quaternion.LookRotation(relativePos);
		transform.rotation = rotation;

        //��������ͬ�ڣ�
		transform.LookAt(target);
	}
