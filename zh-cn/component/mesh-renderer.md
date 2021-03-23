# 自定义几何体渲染器

[MeshRenderer](${book.api}classes/core.meshrenderer.html) 是几何体渲染组件，其数据对象是 [Mesh](${book.api}classes/core.mesh.html)。`Mesh` 是个抽象类，开发者一般使用 [ModelMesh](${book.api}classes/core.modelmesh.html) 或 [BufferMesh](${book.api}classes/core.buffermesh.html)。

- `ModelMesh` 封装了常用的设置顶点数据和索引数据的方法，非常简单易用。开发者若想要快速地去自定义几何体可以使用该类。

- `BufferMesh` 可以自由操作顶点缓冲和索引缓冲数据，以及一些与几何体绘制相关的指令。具备高效、灵活、简洁等特点。开发者如果想高效灵活的实现自定义几何体就可以使用该类。

## ModelMesh

`ModelMesh` 简单易用，可以快速定义网格，可以做一些特殊的效果，使用案例可以参考[示例]()。下面详细介绍一下 `ModelMesh` 的用法

### 使用

ModelMesh 的使用主要分有两个步骤：

#### 1. 设置数据

设置数据的 API 有：

| API            | 说明                   |
| -------------- | ---------------------- |
| setPositions   | 设置顶点坐标           |
| setIndices     | 设置索引数据           |
| setNormals     | 设置逐顶点法线数据     |
| setColors      | 设置逐顶点颜色数据     |
| setTangents    | 设置逐顶点切线         |
| setBoneWeights | 设置逐顶点骨骼权重     |
| setBoneIndices | 设置逐顶点骨骼索引数据 |
| setUVs         | 设置逐顶点 uv 数据     |
|                |                        |

以上方法除了 `setPositions` 不能传 `null`，其它都可以传 `null`，表示无需此项数据。

> 以上的 set 对应都有 get 方法，在设置完后可以调用 `getXXX` 查看设置的数据。

#### 2. 上传数据

设置数据完成后，就可以上传数据了。上传数据也分有两种：数据无需再变更，数据可变更。都是调用 `uploadData` 方法。

##### 2.1 数据无需再变更

当数据无需变更时，我们直接使用 `uploadData(true)` ，表明数据不需要变更，再也不需要把 CPU 数据传递到 GPU 。此时 ModelMesh 里的所有 CPU 缓存都会被清空。意味着 Mesh 的数据已经无法访问了。这种方式可以减少内存的占用。

若再调用 `uploadData` 或是调用 `getPositions/setPositions`、`setNormals/getNormals` 等等方法都会抛出异常。

##### 2.2 数据需要变更

开发者有时需要做一些特殊效果，存在不断变更顶点数据的情况。那我们在调用 `uploadData` 需要传参 `false`。

### 数据变更时的注意事项

在数据频繁变更时，若开发者需要关注一些最佳实践，否则会出现一些性能问题。很重要的一点是尽量**避免顶点 Buffer 的重新创建**。那么我们要做的有：

1. 不要频繁变更顶点数量。
2. 不要频繁变更顶点结构。

若是出现上述两种情况，只能丢弃原有的 Buffer 重新创建，需要整段数据重新赋值。遍历和 GC 的开销相对都比较大。

还有一点是避免**数学库的对象的重新创建**，在我们更改数据的时候，最好是**复用之前的数组和数学库对象**。这里的最佳实践是：

```typescript
// 正确的做法
// 1. 获取顶点对象
const positions = modelMesh.getPositions();
// 2. 更改已有的数据
positions[0].setValue(10, 20, 30);
// 3. 触发顶点数据的变更
modelMesh.setPositions(positions);
// 4. 重新上传数据
modelMesh.uploadData(false);

// 不好的做法:
// 1. 新创建数组
const positions = [new Vector3()];
// 2. 新创建数学库对象
positions[0] = new Vector3(10, 20, 30);
```

## BufferMesh
### 原理图
我们先概览一下 `BufferMesh`  的原理图

![image.png](https://gw.alipayobjects.com/mdn/rms_d27172/afts/img/A*pYFLSIbNKZYAAAAAAAAAAAAAARQnAQ)

`BufferMesh` 有三大核心元素分别是：

|名称|解释|
|:--|:--|
|[VertexBufferBinding](${book.api}classes/core.vertexbufferbinding.html)|顶点缓冲绑定，用于将顶点缓冲和顶点跨度（字节）打包。|
|[VertexElement](${book.api}classes/core.vertexelement.html)|顶点元素，用于描述顶点语义、顶点偏移、顶点格式和顶点缓冲绑定索引等信息。|
|[IndexBufferBinding](${book.api}classes/core.indexbufferbinding.html)|索引缓冲绑定（可选），用于将索引缓冲和索引格式打包。|
  
其中  `IndexBufferBinding`  为可选，也就是说必要核心元素只有两个，分别通过 [`setVertexBufferBindings()`](${book.api}classes/core.primitive.html#setvertexbufferbindings) 接口和 [`setVertexElements()`](${book.api}classes/core.primitive.html#setvertexelements) 接口设置。 最后一项就是通过 [`addSubMesh`](${book.api}classes/core.buffermesh.html#addsubmesh) 添加子 *SubMesh* ，并设置顶点或索引绘制数量， *SubMesh* 包含三个属性分别是起始绘制偏移、绘制数量、图元拓扑，并且开发者可以自行添加多个 *SubMesh*，每个子几何体均可对应独立的材质。


### 常用案例
这里列举几个 [MeshRenderer](${book.api}classes/core.meshrenderer.html) 和 [BufferMesh](${book.api}classes/core.buffermesh.html) 的常用使用场景，因为这个类的功能偏底层和灵活，所以这里给出了比较详细的代码。

#### 交错顶点缓冲
常用方式，比如自定义 Mesh、Particle 等实现，具有显存紧凑，每帧 CPU 数据上传至 GPU 次数少等优势。这个案例的主要特点是多个 [VertexElement](${book.api}classes/core.vertexelement.html) 对应一个 *VertexBuffer* （[Buffer](${book.api}classes/core.buffer.html)），仅使用一个 *VertexBuffer* 就可以将不同顶点元素与 Shader 关联。

```typescript
// add MeshRenderer component
const renderer = entity.addComponent(MeshRenderer);

// create mesh
const mesh = new BufferMesh(engine);

// create vertices.
const vertices = new ArrayBuffer(vertexByteCount);

// create vertexBuffer and upload vertices.
const vertexBuffer = new Buffer(engine, BufferBindFlag.VertexBuffer, vertices);

// bind vertexBuffer with stride, stride is every vertex byte length,so the value is 16.
mesh.setVertexBufferBindings(new VertexBufferBinding(vertexBuffer, 16));

// add vertexElement to tell GPU how to read vertex from vertexBuffer.
mesh.setVertexElements([new VertexElement("POSITION", 0, VertexElementFormat.Vector3, 0),
                            new VertexElement("COLOR", 12, VertexElementFormat.NormalizedUByte4, 0)]);

// add one subMesh and set how many vertex you want to render.
mesh.addSubMesh(0, vertexCount);

// set mesh
renderer.mesh = mesh;
```
#### 独立顶点缓冲
动态顶点 buffer 和静态顶点 buffer 混用时具有优势，比如 *position* 为静态，但 *color* 为动态，独立顶点缓冲可以仅更新颜色数据至 GPU。这个案例的主要特点是一个 [VertexElement](${book.api}classes/core.vertexelement.html) 对应一个 *VertexBuffer* ，可以分别调用 [Buffer](${book.api}classes/core.buffer.html) 对象的 [setData](${book.api}classes/core.buffer.html#setdata) 方法独立更新数据。

```typescript
// add MeshRenderer component
const renderer = entity.addComponent(MeshRenderer);

// create mesh
const mesh = new BufferMesh(engine);

// create vertices.
const positions = new Float32Array(vertexCount);
const colors = new Uint8Array(vertexCount);

// create vertexBuffer and upload vertices.
const positionBuffer = new Buffer(engine, BufferBindFlag.VertexBuffer, positions);
const colorBuffer = new Buffer(engine, BufferBindFlag.VertexBuffer, colors);

// bind vertexBuffer with stride,stride is every vertex byte length,so the value is 12.
mesh.setVertexBufferBindings([new VertexBufferBinding(positionBuffer, 12),
                                 	new VertexBufferBinding(colorBuffer, 4)]);

// add vertexElement to tell GPU how to read vertex from vertexBuffer.
mesh.setVertexElements([new VertexElement("POSITION", 0, VertexElementFormat.Vector3, 0),
                            new VertexElement("COLOR", 0, VertexElementFormat.NormalizedUByte4, 1)]);

// add one subMesh and set how many vertex you want to render.
mesh.addSubMesh(0, vertexCount);

// set mesh
renderer.mesh = mesh;
```


#### Instance 渲染
GPU Instance 渲染是三维引擎的常用技术，比如可以把相同几何体形状的物体一次性渲染到不同的位置，可以大幅提升渲染性能。这个案例的主要特点是使用了 [VertexElement](${book.api}classes/core.vertexelement.html) 的实例功能，其构造函数的最后一个参数表示实例步频（在缓冲中每前进一个顶点绘制的实例数量，非实例元素必须为 0），[BufferMesh](${book.api}classes/core.buffermesh.html) 的 [instanceCount](${book.api}classes/core.buffermesh.html#instancecount) 表示实例数量。

```typescript
// add MeshRenderer component
const renderer = entity.addComponent(MeshRenderer);

// create mesh
const mesh = new BufferMesh(engine);

// create vertices.
const vertices = new ArrayBuffer( vertexByteLength );

// create instance data.
const instances = new Float32Array( instanceDataLength );

// create vertexBuffer and upload vertex data.
const vertexBuffer = new Buffer( engine, BufferBindFlag.VertexBuffer, vertices );

// create instance buffer and upload instance data.
const instanceBuffer = new Buffer( engine, BufferBindFlag.VertexBuffer, instances );

// bind vertexBuffer with stride, stride is every vertex byte length,so the value is 16.
mesh.setVertexBufferBindings([new VertexBufferBinding( vertexBuffer, 16 ),
                                  new VertexBufferBinding( instanceBuffer, 12 )]);

// add vertexElement to tell GPU how to read vertex from vertexBuffer.
mesh.setVertexElements([new VertexElement( "POSITION", 0, VertexElementFormat.Vector3, 0 ),
                            new VertexElement( "COLOR", 12, VertexElementFormat.NormalizedUByte4, 0 ),
                            new VertexElement( "INSTANCE_OFFSET", 0, VertexElementFormat.Vector3, 1 , 1 ),
                            new VertexElement( "INSTANCE_ROTATION", 12, VertexElementFormat.Vector3, 1 , 1 )]]);

// add one sub mesh and set how many vertex you want to render, here is full vertexCount.
mesh.addSubMesh(0, vertexCount);

// set mesh
renderer.mesh = mesh;
```


### 索引缓冲
使用索引缓冲可以复用顶点缓冲内的顶点，从而达到节省显存的目的。其使用方式很简单，就是在原基础上增加了索引缓冲对象，以下代码是在第一个 **交错顶点缓冲** 案例的基础上修改而来的

```typescript
// add MeshRenderer component
const renderer = entity.addComponent(MeshRenderer);

// create mesh
const mesh = new BufferMesh(engine);

// create vertices.
const vertices = new ArrayBuffer(vertexByteCount);

// create indices.
const indices = new Uint16Array(indexCount);

// create vertexBuffer and upload vertices.
const vertexBuffer = new Buffer(engine, BufferBindFlag.VertexBuffer, vertices);

// create indexBuffer and upload indices.
const indexBuffer = new Buffer(engine, BufferBindFlag.IndexBuffer, indices);

// bind vertexBuffer with stride, stride is every vertex byte length,so the value is 16.
mesh.setVertexBufferBindings(new VertexBufferBinding(vertexBuffer, 16));

// bind vertexBuffer with format.
mesh.setIndexBufferBinding(indexBuffer, IndexFormat.UInt16);

// add vertexElement to tell GPU how to read vertex from vertexBuffer.
mesh.setVertexElements([new VertexElement("POSITION", 0, VertexElementFormat.Vector3, 0),
                            new VertexElement("COLOR", 12, VertexElementFormat.NormalizedUByte4, 0)]);

// add one subMesh and set how many vertex you want to render.
mesh.addSubMesh(0, vertexCount);

// set mesh
renderer.mesh = mesh;
```