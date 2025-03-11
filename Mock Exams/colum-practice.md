### JsonPath + jq
https://kubernetes.io/docs/reference/kubectl/jsonpath/

以下結合-o json 然後用jq -r 擷取欄位

controlplane:~$ k get deploy deployment1 -o json | jq -r '.spec.template.spec.containers'
[
  {
    "image": "busybox",
    "imagePullPolicy": "Always",
    "name": "busybox",
    "resources": {},
    "terminationMessagePath": "/dev/termination-log",
    "terminationMessagePolicy": "File"
  }
]
controlplane:~$ k get deploy deployment1 -o json | jq -r '.spec.template.spec.containers[]'
{
  "image": "busybox",
  "imagePullPolicy": "Always",
  "name": "busybox",
  "resources": {},
  "terminationMessagePath": "/dev/termination-log",
  "terminationMessagePolicy": "File"
}
controlplane:~$ k get deploy deployment1 -o json | jq -r '.spec.template.spec.containers[].images'
null
controlplane:~$ k get deploy deployment1 -o json | jq -r '.spec.template.spec.containers[].image'
busybox
controlplane:~$ 


controlplane:~$ kubectl get deploy -o json | jq -r '.items[] | "\(.metadata.name) - \(.spec.template.spec.containers[].image)"'
deployment1 - busybox

items[]: 用來遍歷所有對象的陣列


### Custom-Columns
https://kubernetes.io/docs/reference/kubectl/


controlplane:~$ k get deploy deployment1 -o json | jq -r '.spec.template.spec.containers[].image'
busybox

(跟-o json 結合jq 指令差不多)

kubectl get deploy deployment1 -o custom-columns=Images:.spec.template.spec.containers[].image
Images
busybox


在 **CKA/CKAD 考試** 時，如果需要從 `kubectl get ... -o <format>` 中提取資訊，**建議使用 `json + jq`**，而不是 `custom-columns`。這是因為：



### **✅ 為何 `json + jq` 更適合考試？**
1. **更靈活，適用於多層級 JSON 結構**
   - `jq` 可以處理 **巢狀 JSON 結構**，而 `custom-columns` 無法處理複雜的 JSON。
   - 例如，要獲取 **所有 Pod 的 images**：
     ```sh
     kubectl get pods -o json | jq -r '.items[].spec.containers[].image'
     ```
     - `jq` 會自動展開 `items[]` 和 `containers[]`，適用於多個 Pod 和多個 containers 的情況。

2. **適用於 `select()` 篩選**
   - 如果你想篩選某些條件（如只列出 `nginx` 容器），`jq` 可以用 `select()`：
     ```sh
     kubectl get pods -o json | jq -r '.items[] | select(.spec.containers[].image | test("nginx")).metadata.name'
     ```
   - `custom-columns` 無法做到這樣的篩選。

3. **更容易格式化輸出**
   - `jq` 允許自訂輸出格式，例如：
     ```sh
     kubectl get deploy -o json | jq -r '.items[] | "\(.metadata.name) - \(.spec.template.spec.containers[].image)"'
     ```
   - 這樣你可以獲取 Deployment 名稱和對應的 image，類似：
     ```
     deployment1 - busybox
     deployment2 - nginx
     ```

**❌ `custom-columns` 的問題**
- **不能處理陣列（array）**：如果 `spec.template.spec.containers` 是陣列，`custom-columns` 無法直接解析，可能會出現 `<none>`。
- **不支持篩選 (`select()`)**：如果你想篩選某些條件，例如只顯示特定的 `image`，`custom-columns` 無法辦到。
- **無法處理巢狀結構（nested JSON）**：如果資料結構比較深，`custom-columns` 可能無法提取你要的資訊。

示例：
```sh
kubectl get deploy -o custom-columns=NAME:.metadata.name,IMAGE:.spec.template.spec.containers[].image
```
- 這個命令**只能適用於單層級 JSON**，但無法處理 **巢狀結構**，如 `.items[]` 這樣的陣列。


**📝 考試時的最佳策略**
| 方式 | 適用情境 | 推薦度 |
|------|---------|------|
| **`kubectl get ... -o json 跟 jq`** | 需要處理巢狀結構、多個 Pod、篩選數據時 | ⭐⭐⭐⭐⭐ |
| **`kubectl get ... -o custom-columns`** | 只需快速列出少量已知字段 | ⭐⭐ |
| **`kubectl get ... -o jsonpath`** | 只提取單個值，無需複雜篩選 | ⭐⭐⭐ |

