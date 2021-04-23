# Unity-CSharp-Optimize-Guildline

## 盡可能的讓判斷條件不要在迴圈中
調整前
```
void Update()  
{  
    for (int i = 0; i < myArray.Length; i++)  
    {  
        if (exampleBool)  
        {  
            ExampleFunction(myArray[i]);  
        }  
    }  
}  
```

調整後
```
void Update()  
{  
    if (exampleBool)  
    {  
        for (int i = 0; i < myArray.Length; i++)  
        {  
            ExampleFunction(myArray[i]);  
        }  
    }  
}  
```

## 只有在資料改變時在執行方法
### 案例1
調整前
```
private int m_score;  
  
public void IncrementScore(int incrementBy)  
{  
    m_score += incrementBy;  
}  
  
void Update()  
{  
    DisplayScore(m_score);  
}  
```

調整後
```
private int m_score;  
  
public void IncrementScore(int incrementBy)  
{  
    m_score += incrementBy;  
    DisplayScore(m_score);  
}  
```

### 案例2
調整前
```
void Update()  
{  
    ExampleGarbageGeneratingFunction(transform.position.x);  
}  
```

調整後
```
private float m_previousTransformPositionX;  
  
void Update()  
{  
    float transformPositionX = transform.position.x;  
    if (transformPositionX != m_previousTransformPositionX)  
    {  
        ExampleGarbageGeneratingFunction(transformPositionX);  
        m_previousTransformPositionX = transformPositionX;  
    }  
}  
```

## 避免逐幀計算 Time Slicing
調整前
```
void Update()  
{  
    ExampleExpensiveFunction();  
}  
```

調整後
```
private int m_interval = 3;  
  
void Update()  
{  
    if (Time.frameCount % m_interval == 0)  
    {  
        ExampleExpensiveFunction();  
    }  
}  
```

## 使用快取
應盡量避免生成參考類型物件，即任何 new XXXXX()，否則會導致記憶體分配

### 案例 GetComponent
調整前
```
void Update()  
{  
    Renderer myRenderer = GetComponent<Renderer>();  
    ExampleFunction(myRenderer);  
}  
```

調整後
```
private Renderer m_renderer;  
  
void Start()  
{  
    m_renderer = GetComponent<Renderer>();  
}  
  
void Update()  
{  
    ExampleFunction(m_renderer);  
}  
```

### 案例 FindObjectsOfType
調整前
```
void OnTriggerEnter(Collider other)  
{  
    Renderer[] renderers = FindObjectsOfType<Renderer>();  
    ExampleFunction(renderers);  
}  
```

調整後
```
private Renderer[] m_renderers;  
  
void Start()  
{  
    m_renderers = FindObjectsOfType<Renderer>();  
}  
  
void OnTriggerEnter(Collider other)  
{  
    ExampleFunction(m_renderers);  
}  
```

### 案例 new List()
調整前
```
void Update()  
{  
    List myList = new List();  
    PopulateList(myList);  
}  
```

調整後
```
private List m_myList = new List();  
void Update()  
{  
    m_myList.Clear();  
    PopulateList(m_myList);  
}  
```

### 案例 new WaitForSeconds
調整前
```
while (!isComplete)  
{  
    yield return new WaitForSeconds(1f);  
}  
```

調整後
```
WaitForSeconds delay = new WaitForSeconds(1f);  
  
while (!isComplete)  
{  
    yield return delay;  
}  
```

### 案例 transform
調整前
```
public void UpdateCharacter()  
{  
    var lastPos = transform.position;  
    transform.position = lastPos  
        + wantedVelocity * (speed * speedFactor  
        * Mathf.Sin(someOtherFactor)  
        * drag * friction * Time.deltaTime);  
}  
```

調整後
```
private Transform m_transform;  
  
private void Awake()  
{  
    m_transform = transform;  
}  
  
public void UpdateCharacter()  
{  
    var lastPos = m_transform.position;  
    m_transform.position = lastPos  
        + wantedVelocity * (speed * speedFactor  
        * Mathf.Sin(someOtherFactor)  
        * drag * friction * Time.deltaTime);  
}  
```

### 案例 Time.deltaTime
調整前
```
public void UpdateCharacter()  
{  
    float factor = speed * speedFactor  
        * Mathf.Sin(someOtherFactor)  
        * drag * friction * Time.deltaTime;  
  
    m_lastPos.x += wantedVelocity.x * factor;  
    m_lastPos.y += wantedVelocity.y * factor;  
    m_lastPos.z += wantedVelocity.z * factor;  
    m_transform.localPosition = m_lastPos;  
}  
```

調整後
```
private float m_deltaTime;  
  
public void UpdateCharacter()  
{  
    float factor = speed * speedFactor  
        * Mathf.Sin(someOtherFactor)  
        * drag * friction * m_deltaTime;  
  
    m_lastPos.x += wantedVelocity.x * factor;  
    m_lastPos.y += wantedVelocity.y * factor;  
    m_lastPos.z += wantedVelocity.z * factor;  
    m_transform.localPosition = m_lastPos;  
}  
```

## 避免使用 LINQ
雖然 LINQ 簡潔易讀寫，但通常需要更多計算及記憶體配置

## 避免使用下列 Unity API
GameObject.SendMessage
GameObject.BroadcastMessage
UnityEngine.Object.Find()
UnityEngine.Object.FindWithTag()
UnityEngine.Object.FindObjectOfType()
UnityEngine.Object.FindObjectsOfType()
UnityEngine.Object.FindGameObjectsWithTag()
Camera.main

## 使用 GameObject.CompareTag 取代 GameObject.tag
調整前
```
private string m_playerTag = "Player";  
  
void OnTriggerEnter(Collider other)  
{  
    bool isPlayer = other.gameObject.tag == m_playerTag;  
}  
```

調整後
```
private string m_playerTag = "Player";  
  
void OnTriggerEnter(Collider other)  
{  
    bool isPlayer = other.gameObject.CompareTag(m_playerTag);  
}  
```

## 使用 yield return null 取代 yield return 0
調整前
```
yield return 0;  
```

調整後
```
yield return null; 
```

## 減少 Vector 計算
### 案例1
調整前
```
public void UpdateCharacter()  
{  
    var lastPos = transform.position;  
    transform.position = lastPos  
        + wantedVelocity * speed * speedFactor  
        * Mathf.Sin(someOtherFactor)  
        * drag * friction * Time.deltaTime;  
}  
```

調整後
```
public void UpdateCharacter()  
{  
    var lastPos = transform.position;  
    transform.position = lastPos  
        + wantedVelocity * (speed * speedFactor  
        * Mathf.Sin(someOtherFactor)  
        * drag * friction * Time.deltaTime);  
}  
```

### 案例2
調整前
```
public void UpdateCharacter()  
{  
    m_lastPos += wantedVelocity * (speed * speedFactor  
        * Mathf.Sin(someOtherFactor)  
        * drag * friction * Time.deltaTime);  
    m_transform.localPosition = m_lastPos;  
}  
```

調整後
```
public void UpdateCharacter()  
{  
    float factor = speed * speedFactor  
        * Mathf.Sin(someOtherFactor)  
        * drag * friction * Time.deltaTime;  
  
    m_lastPos.x += wantedVelocity.x * factor;  
    m_lastPos.y += wantedVelocity.y * factor;  
    m_lastPos.z += wantedVelocity.z * factor;  
    m_transform.localPosition = m_lastPos;  
}  
```

## 盡可能使用 Transform.localPosition
調整前
```
public void UpdateCharacter()  
{  
    var lastPos = m_transform.position;  
    m_transform.position = lastPos  
        + wantedVelocity * (speed * speedFactor  
        * Mathf.Sin(someOtherFactor)  
        * drag * friction * Time.deltaTime);  
}  
```

調整後
```
public void UpdateCharacter()  
{  
    var lastPos = m_transform.localPosition;  
    m_transform.localPosition= lastPos  
        + wantedVelocity * (speed * speedFactor  
        * Mathf.Sin(someOtherFactor)  
        * drag * friction * Time.deltaTime);  
}  
```

## 減少取得 Transform.position、Transform.localPosition
調整前
```
public void UpdateCharacter()  
{  
    var lastPos = m_transform.localPosition;  
    m_transform.localPosition = lastPos  
        + wantedVelocity * (speed * speedFactor  
        * Mathf.Sin(someOtherFactor)  
        * drag * friction * Time.deltaTime);  
}  
```

調整後
```
private Vector3 m_lastPos = Vector3.zero;  
  
public void UpdateCharacter()  
{  
    m_lastPos += wantedVelocity * (speed * speedFactor  
        * Mathf.Sin(someOtherFactor)  
        * drag * friction * Time.deltaTime);  
    m_transform.localPosition = m_lastPos;  
}  
```

## 避免使用 foreach
調整前
```
foreach(int value in m_list)  
{  
    DoSomething(value);  
}  
```

調整後
```
int length = m_list.Count;  
for(int i = 0; i < length; i++)  
{  
    DoSomething(m_list[i]);  
}  
```

### 效能差異
執行次數 100000
|         | Time ms | GC Alloc |
|---------|---------|----------|
| foreach | 28.04   | 48 B     |
| for     | 17.47   | 0 B      |

## 盡量使用 Array 取代 List
調整前
```
int length = m_list.Count;  
for(int i = 0; i < length; i++)  
{  
    DoSomething(m_list[i]);  
}  
```

調整後
```
int length = m_array.Length;  
for (int i = 0; i < length; i++)  
{  
    DoSomething(m_array[i]);  
}  
```

### 效能差異
執行次數 100000
|       | Time ms | GC Alloc |
|-------|---------|----------|
| List  | 17.44   | 0 B      |
| Array | 8.70    | 0 B      |

## 使用 FastEnum 取代 Enum
調整前
```
Enum.GetName(typeof(State), stringValue);  
Enum.GetName(typeof(State), intValue);  
Enum.GetNames(typeof(State));  
Enum.GetValues(typeof(State));  
Enum.Parse(typeof(State), stringValue);  
 
State result;  
Enum.TryParse<State>(stringValue, out result);  
 
result.ToString();
```

調整後
```
FastEnum.GetName<State>(stringValue);  
FastEnum.GetName<State>(intValue);  
FastEnum.GetNames<State>();  
FastEnum.GetValues<State>();  
FastEnum.Parse<State>(stringValue); 
 
State result;  
FastEnum.TryParse<State>(stringValue, out result);  
 
FastEnum.ToString<State>((int)result); 
```

效能差異
執行次數 100000
Enum > FastEnum
|                 | Time ms         | GC Alloc        |
|-----------------|-----------------|-----------------|
| GetName(string) | 339.60 > 18.30  | 1.9 MB > 0 B    |
| GetName(int)    | 662.15 > 18.56  | 3.8 MB > 0 B    |
| GetNames        | 227.57 > 12.99  | 3.8 MB > 0 B    |
| GetValues       | 821.96 > 15.63  | 8.8 MB > 1.0 KB |
| Parse           | 765.91 > 184.60 | 12.2 MB > 0 B   |
| TryParse        | 761.73 > 185.73 | 12.2 MB > 0 B   |
| ToString        | 579.02 > 18.10  | 3.8 MB > 0 B    |

## 屬性 Property
屬性 Property 底層依然是透過方法實現
使用 Property 會導致執行效能較差，但使用上及維護上會較為方便，能夠針對 get 或 set 設置不同的訪問層級和檢查機制，且支援任何方法的語言特性，如 virtual、abstract

調整前
```
public int intValue { get; set; }  
```

調整後
```
public int intValue;  
```

### 效能差異
執行次數 1000000
|          | Time ms | GC Alloc |
|----------|---------|----------|
| Property | 41.00   | 0 B      |
| Field    | 0       | 0 B      |

## 使用 is 或 as 而不是強制類型轉換
is: 能夠檢查一個對象是否兼容於其他指定類型，回傳一個 Bool 值且不會跳出異常
as: 作用跟強制類型轉換一樣，但不會跳出異常，如果轉換失敗，會回傳 null

## 避免使用大量 MonoBehaviour
由於 Unity MonoBehaviour 使用的是 Messaging System，能夠讓開發者在 MonoBehaviour 中自行定義特殊方法，如: Awake、Start、Update、FixedUpdate、LateUpdate 等。
當有使用大量 MonoBehaviour.Update、FixedUpdate、LateUpdate 需求時，應使用 CoreComponent 取代。

調整前
```
using UnityEngine;  
public class TestMonoBehaviour : MonoBehaviour  
{  
    private int m_count;  
    private void Update()  
    {  
        m_count++;  
    }  
}  
```

調整後
```
using JSLCore;  
public class TestOptimizedMonoBehaviour : CoreComponent 
{  
    private int m_count;  
    public override void Tick(float deltaTime, float unscaledDeltaTime)  
    {  
        m_count++;  
    }  
}  
```

### 效能差異
執行次數 10000
|               | Time ms | GC Alloc |
|---------------|---------|----------|
| MonoBehaviour | 4.01    | 0 B      |
| CoreComponent | 1.82    | 0 B      |

## 大量字元串接時使用 String.Concat
調整前
```
char c = 'X';  
string output = string.Empty;  
for(int i = 0; i < stringLength; i++)  
{  
    output += c;  
}  
```

調整後
```
char c = 'X';  
char[] chars = new char[stringLength];  
for (int i = 0; i < stringLength; i++)  
{  
    chars[i] = c;  
}  
  
string output = string.Concat(chars);  
```

### 效能差異
執行次數 100000
|               | Time ms  | GC Alloc |
|---------------|----------|----------|
| String +=     | 11628.02 | 1.32 GB  |
| String.Concat | 112.28   | 3.3 MB   |

## 生成大量相同物件使用 Object Pool
調整前
```
GameObject instance = GameObject.Instantiate(m_prefab);  
GameObject.Destroy(instance);  
```

調整後
```
GameObject instance = PoolManager.Instance.Get(m_prefab);  
PoolManager.Instance.Destroy(instance);  
```

## 使用 struct 取代 class
調整前
```
class VectorClass  
{  
    public int X { get; set; }  
    public int Y { get; set; }  
}  
  
private void Execute()  
{  
    VectorClass[] vectors = new VectorClass[10000];  
    for (int i = 0; i < vectors.Length; i++)  
    {  
        vectors[i] = new VectorClass();  
        vectors[i].X = 5;  
        vectors[i].Y = 10;  
    }  
}  
```

調整後
```
struct VectorStruct  
{  
    public int X { get; set; }  
    public int Y { get; set; }  
}  
  
private void Execute()  
{  
    VectorStruct[] vectors = new VectorStruct[10000];  
    for (int i = 0; i < vectors.Length; i++)  
    {  
        vectors[i].X = 5;  
        vectors[i].Y = 10;  
    }  
}  
```

### 效能差異
執行次數 10000
|        | Time ms | GC Alloc |
|--------|---------|----------|
| class  | 5.68    | 312.5 KB |
| struct | 2.00    | 78.2 KB  |

## 避免使用解構子
調整前
```
class SimpleWithFinalizer  
{  
    public int x { get; set; }  
  
    ~SimpleWithFinalizer()  
    {  
  
    }  
}  
  
private void Execute()  
{  
    for (int i = 0; i < 10000; i++)  
    {  
        new SimpleWithFinalizer();  
    }  
}  
```

調整後
```
class Simple  
{  
    public int x { get; set; }  
}  
  
private void Execute()  
{  
    for (int i = 0; i < 10000; i++)  
    {  
        new Simple();  
    }  
}  
```

### 效能差異
執行次數 10000
|            | Time ms | GC Alloc |
|------------|---------|----------|
| 沒有解構子 | 2.14    | 195.3 KB |
| 有解構子   | 4.48    | 195.3 KB |

## 資料來源
[Fixing Performance Problems][https://learn.unity.com/tutorial/fixing-performance-problems#5c7f8528edbc2a002053b595]
[對 Unity 的效能建議 - Mixed Reality][https://docs.microsoft.com/zh-tw/windows/mixed-reality/develop/unity/performance-recommendations-for-unity]
[Unite 2016 - Tools, Tricks and Technologies for Reaching Stutter Free 60 FPS in INSIDE][https://www.youtube.com/watch?v=mQ2KTRn4BMI]
[10000 Update() calls][https://blogs.unity3d.com/2015/12/23/1k-update-calls/]
[8 Techniques to Avoid GC Pressure and Improve Performance in C# .NET][https://michaelscodingspot.com/avoid-gc-pressure/]
