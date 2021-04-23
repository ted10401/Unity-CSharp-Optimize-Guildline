Unity-CSharp-Optimize-Guildline
=============================

- [盡可能的讓判斷條件不要在迴圈中](#盡可能的讓判斷條件不要在迴圈中)
- [只有在資料改變時在執行方法](#只有在資料改變時在執行方法)
- [避免逐幀計算](#避免逐幀計算)
- [使用快取](#使用快取)
- [避免使用 LINQ](#避免使用-linq)
- [避免使用下列 Unity API](#避免使用下列-unity-api)
- [使用 GameObject.CompareTag 取代 GameObject.tag](#使用-gameobject.comparetag-取代-gameobject.tag)
- [使用 yield return null 取代 yield return 0](#使用-yield-return-null-取代-yield-return-0)
- [減少 Vector 計算](#減少-vector-計算)
- [盡可能使用 Transform.localPosition](#盡可能使用-transform.localposition)
- [減少取得 Transform.position、Transform.localPosition](#減少取得-transform.position、transform.localposition)
- [避免使用 foreach](#避免使用-foreach)
- [盡量使用 Array 取代 List](#盡量使用-array-取代-list)
- [避免使用 Enum 函式](#避免使用-enum-函式)
- [避免使用屬性 Property](#避免使用屬性-property)
- [使用 is 或 as 而不是強制類型轉換](#使用-is-或-as-而不是強制類型轉換)
- [避免大量使用 MonoBehaviour.Update、FixedUpdate、LateUpdate](#避免大量使用-monobehaviour.update、fixedupdate、lateupdate)
- [大量字元串接時使用 String.Concat](#大量字元串接時使用-string.concat)
- [生成大量相同物件使用 Object Pool](#生成大量相同物件使用-object-pool)
- [使用 struct 取代 class](#使用-struct-取代-class)
- [避免使用解構子](#避免使用解構子)
- [資料來源](#資料來源)

# 盡可能的讓判斷條件不要在迴圈中
調整前
```csharp
private void Update()  
{  
    for (int i = 0; i < array.Length; i++)  
    {  
        if (exampleBool)  
        {  
            ExampleFunction(array[i]);  
        }  
    }  
}  
```

調整後
```csharp
private void Update()  
{  
    if (exampleBool)  
    {  
        for (int i = 0; i < array.Length; i++)  
        {  
            ExampleFunction(array[i]);  
        }  
    }  
}  
```

# 只有在資料改變時在執行方法
## 案例1
調整前
```csharp
private int m_score;  
  
public void AddScore(int value)  
{  
    m_score += value;  
}  
  
private void Update()  
{  
    DisplayScore(m_score);  
}  
```

調整後
```csharp
private int m_score;  
  
public void AddScore(int value)  
{  
    m_score += value;  
    DisplayScore(m_score);  
}  
```

## 案例2
調整前
```csharp
private void Update()  
{  
    ExampleGarbageGeneratingFunction(transform.position.x);  
}  
```

調整後
```csharp
private float m_curPosX;
private float m_lastPosX;  
  
private void Update()  
{  
    m_curPosX = transform.position.x;  
    if (m_lastPosX != m_curPosX)  
    {
        m_lastPosX = m_curPosX;
        ExampleGarbageGeneratingFunction(m_lastPosX);  
    }  
}  
```

# 避免逐幀計算
調整前
```csharp
private void Update()  
{  
    ExampleExpensiveFunction();  
}  
```

調整後
```csharp
private int m_interval = 3;  
  
private void Update()  
{  
    if (Time.frameCount % m_interval == 0)  
    {  
        ExampleExpensiveFunction();  
    }  
}  
```

# 使用快取
應盡量避免生成參考類型物件，即任何 new XXXXX()，否則會導致記憶體分配

## 案例 GetComponent
調整前
```csharp
private void Update()  
{  
    Renderer renderer = GetComponent<Renderer>();  
    ExampleFunction(renderer);  
}  
```

調整後
```csharp
private Renderer m_renderer;  
  
private void Awake()  
{  
    m_renderer = GetComponent<Renderer>();  
}  
  
private void Update()  
{  
    ExampleFunction(m_renderer);  
}  
```

## 案例 FindObjectsOfType
調整前
```csharp
private void OnTriggerEnter(Collider other)  
{  
    Renderer[] renderers = FindObjectsOfType<Renderer>();  
    ExampleFunction(renderers);  
}  
```

調整後
```csharp
private Renderer[] m_renderers;  
  
private void Awake()  
{  
    m_renderers = FindObjectsOfType<Renderer>();  
}  
  
private void OnTriggerEnter(Collider other)  
{  
    ExampleFunction(m_renderers);  
}  
```

## 案例 List
調整前
```csharp
private void Update()  
{  
    List list = new List();  
    PopulateList(list);  
}  
```

調整後
```csharp
private List m_list = new List();  
void Update()  
{  
    m_list.Clear();  
    PopulateList(m_list);  
}  
```

## 案例 new WaitForSeconds
調整前
```csharp
while (!isDone)  
{  
    yield return new WaitForSeconds(1f);  
}  
```

調整後
```csharp
WaitForSeconds delay = new WaitForSeconds(1f);  
  
while (!isDone)  
{  
    yield return delay;  
}  
```

## 案例 transform
調整前
```csharp
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
```csharp
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

## 案例 Time.deltaTime
調整前
```csharp
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
```csharp
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

# 避免使用 LINQ
雖然 LINQ 簡潔易讀寫，但通常需要更多計算及記憶體配置

# 避免使用下列 Unity API
- [GameObject.SendMessage](https://docs.unity3d.com/ScriptReference/GameObject.SendMessage.html)
- [GameObject.BroadcastMessage](https://docs.unity3d.com/ScriptReference/GameObject.BroadcastMessage.html)
- [GameObject.Find](https://docs.unity3d.com/ScriptReference/GameObject.Find.html)
- [GameObject.FindGameObjectsWithTag](https://docs.unity3d.com/ScriptReference/GameObject.FindGameObjectsWithTag.html)
- [GameObject.FindWithTag](https://docs.unity3d.com/ScriptReference/GameObject.FindWithTag.html)
- [Object.FindObjectOfType](https://docs.unity3d.com/ScriptReference/Object.FindObjectOfType.html)
- [Object.FindObjectsOfType](https://docs.unity3d.com/ScriptReference/Object.FindObjectsOfType.html)
- [Camera.main](https://docs.unity3d.com/ScriptReference/Camera-main.html)

# 使用 GameObject.CompareTag 取代 GameObject.tag
調整前
```csharp
private const string TAG_PLAYER = "Player";  
  
void OnTriggerEnter(Collider other)  
{  
    bool isPlayer = other.gameObject.tag == TAG_PLAYER;  
}  
```

調整後
```csharp
private const string TAG_PLAYER = "Player";  
  
void OnTriggerEnter(Collider other)  
{  
    bool isPlayer = other.gameObject.CompareTag(TAG_PLAYER);  
}  
```

# 使用 yield return null 取代 yield return 0
調整前
```csharp
yield return 0;  
```

調整後
```csharp
yield return null; 
```

# 減少 Vector 計算
## 案例1
調整前
```csharp
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
```csharp
public void UpdateCharacter()  
{  
    var lastPos = transform.position;  
    transform.position = lastPos  
        + wantedVelocity * (speed * speedFactor  
        * Mathf.Sin(someOtherFactor)  
        * drag * friction * Time.deltaTime);  
}  
```

## 案例2
調整前
```csharp
public void UpdateCharacter()  
{  
    m_lastPos += wantedVelocity * (speed * speedFactor  
        * Mathf.Sin(someOtherFactor)  
        * drag * friction * Time.deltaTime);  
    m_transform.localPosition = m_lastPos;  
}  
```

調整後
```csharp
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

# 盡可能使用 Transform.localPosition
調整前
```csharpcsharp
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
```csharp
public void UpdateCharacter()  
{  
    var lastPos = m_transform.localPosition;  
    m_transform.localPosition= lastPos  
        + wantedVelocity * (speed * speedFactor  
        * Mathf.Sin(someOtherFactor)  
        * drag * friction * Time.deltaTime);  
}  
```

# 減少取得 Transform.position、Transform.localPosition
調整前
```csharp
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
```csharp
private Vector3 m_lastPos = Vector3.zero;  
  
public void UpdateCharacter()  
{  
    m_lastPos += wantedVelocity * (speed * speedFactor  
        * Mathf.Sin(someOtherFactor)  
        * drag * friction * Time.deltaTime);  
    m_transform.localPosition = m_lastPos;  
}  
```

# 避免使用 foreach
調整前
```csharp
foreach(int value in m_list)  
{  
    DoSomething(value);  
}  
```

調整後
```csharp
int length = m_list.Count;  
for(int i = 0; i < length; i++)  
{  
    DoSomething(m_list[i]);  
}  
```

效能比較 (執行次數 100000)
|         | Time ms | GC Alloc |
|---------|---------|----------|
| foreach | 28.04   | 48 B     |
| for     | 17.47   | 0 B      |

# 盡量使用 Array 取代 List
調整前
```csharp
int length = m_list.Count;  
for(int i = 0; i < length; i++)  
{  
    DoSomething(m_list[i]);  
}  
```

調整後
```csharp
int length = m_array.Length;  
for (int i = 0; i < length; i++)  
{  
    DoSomething(m_array[i]);  
}  
```

效能比較 (執行次數 100000)
|       | Time ms | GC Alloc |
|-------|---------|----------|
| List  | 17.44   | 0 B      |
| Array | 8.70    | 0 B      |

# 避免使用 Enum 函式
調整前
```csharp
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
```csharp
FastEnum.GetName<State>(stringValue);  
FastEnum.GetName<State>(intValue);  
FastEnum.GetNames<State>();  
FastEnum.GetValues<State>();  
FastEnum.Parse<State>(stringValue); 
 
State result;  
FastEnum.TryParse<State>(stringValue, out result);  
 
FastEnum.ToString<State>((int)result); 
```

效能比較 (執行次數 100000)<br/>
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

# 避免使用屬性 Property
屬性 Property 底層依然是透過方法實現，使用 Property 會導致執行效能較差，但使用上及維護上會較為方便，能夠針對 get 或 set 設置不同的訪問層級和檢查機制，且支援任何方法的語言特性，如 virtual、abstract。

調整前
```csharp
public int intValue { get; set; }  
```

調整後
```csharp
public int intValue;  
```

效能比較 (執行次數 1000000)
|          | Time ms | GC Alloc |
|----------|---------|----------|
| Property | 41.00   | 0 B      |
| Field    | 0       | 0 B      |

# 使用 is 或 as 而不是強制類型轉換
is: 能夠檢查一個對象是否兼容於其他指定類型，回傳一個 Bool 值且不會跳出異常<br/>
as: 作用跟強制類型轉換一樣，但不會跳出異常，如果轉換失敗，會回傳 null

# 避免大量使用 MonoBehaviour.Update、FixedUpdate、LateUpdate
由於 Unity MonoBehaviour 使用的是 Messaging System，能夠讓開發者在 MonoBehaviour 中自行定義特殊方法，如: Awake、Start、Update、FixedUpdate、LateUpdate 等。<br/>
當有使用大量 MonoBehaviour.Update、FixedUpdate、LateUpdate 需求時，應自定義功能取代。

調整前
```csharp
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
```csharp
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

效能比較 (執行次數 10000)
|               | Time ms | GC Alloc |
|---------------|---------|----------|
| MonoBehaviour | 4.01    | 0 B      |
| CoreComponent | 1.82    | 0 B      |

# 大量字元串接時使用 String.Concat
調整前
```csharp
char c = 'X';  
string output = string.Empty;  
for(int i = 0; i < stringLength; i++)  
{  
    output += c;  
}  
```

調整後
```csharp
char c = 'X';  
char[] chars = new char[stringLength];  
for (int i = 0; i < stringLength; i++)  
{  
    chars[i] = c;  
}  
  
string output = string.Concat(chars);  
```

效能比較 (執行次數 100000)
|               | Time ms  | GC Alloc |
|---------------|----------|----------|
| String +=     | 11628.02 | 1.32 GB  |
| String.Concat | 112.28   | 3.3 MB   |

# 生成大量相同物件使用 Object Pool
調整前
```csharp
GameObject instance = GameObject.Instantiate(m_prefab);  
GameObject.Destroy(instance);  
```

調整後
```csharp
GameObject instance = PoolManager.Instance.Get(m_prefab);  
PoolManager.Instance.Recycle(instance);  
```

# 使用 struct 取代 class
調整前
```csharp
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
```csharp
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

效能比較 (執行次數 10000)
|        | Time ms | GC Alloc |
|--------|---------|----------|
| class  | 5.68    | 312.5 KB |
| struct | 2.00    | 78.2 KB  |

# 避免使用解構子
調整前
```csharp
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
```csharp
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

效能比較 (執行次數 10000)
|            | Time ms | GC Alloc |
|------------|---------|----------|
| 沒有解構子 | 2.14    | 195.3 KB |
| 有解構子   | 4.48    | 195.3 KB |

# 資料來源
- [Fixing Performance Problems](https://learn.unity.com/tutorial/fixing-performance-problems#5c7f8528edbc2a002053b595)
- [對 Unity 的效能建議 - Mixed Reality](https://docs.microsoft.com/zh-tw/windows/mixed-reality/develop/unity/performance-recommendations-for-unity)
- [Unite 2016 - Tools, Tricks and Technologies for Reaching Stutter Free 60 FPS in INSIDE](https://www.youtube.com/watch?v=mQ2KTRn4BMI)
- [10000 Update() calls](https://blogs.unity3d.com/2015/12/23/1k-update-calls/)
- [8 Techniques to Avoid GC Pressure and Improve Performance in C# .NET](https://michaelscodingspot.com/avoid-gc-pressure/)
