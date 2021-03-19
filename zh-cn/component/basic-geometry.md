# 基础网格渲染器

目前提供了以下几种形状的几何体：

- [Cuboid](${book.api}classes/core.primitivemesh.html#createcylinder#createcuboid) **立方体**

```typescript
let sphere = rootEntity.createChild('sphere');
let sphereRenderer = sphere.addComponent(MeshRenderer);
sphereRenderer.mesh = PrimitiveMesh.createCuboid(engine, 2, 2, 2);

// 创建材质
let mtl = new BlinnPhongMaterial(engine, 'test_mtl2', false);
mtl.ambient = new Vector4(0.75, 0.25, 0.25, 1);
mtl.shininess = 100;
sphereRenderer.material = mtl;
```

- [Sphere](${book.api}classes/core.primitivemesh.html#createcylinder#createsphere) **球体**

```typescript
let sphere = rootEntity.createChild('sphere');
let sphereRenderer = sphere.addComponent(MeshRenderer);
sphereRenderer.mesh = PrimitiveMesh.createSphere(engine, 3, 32, 32);

// 创建材质
let mtl = new BlinnPhongMaterial((engine, 'test_mtl2', false);
mtl.ambient = new Vector4(0.75, 0.25, 0.25, 1);
mtl.shininess = 100;
sphereRenderer.material = mtl;
```

- [Plane](${book.api}classes/core.primitivemesh.html#createcylinder#createplane) **平面**

```typescript
// 创建材质
let mtl = new BlinnPhongMaterial(engine, 'test_mtl2', false);
mtl.ambient = new Vector4(0.75, 0.25, 0.25, 1);
mtl.shininess = 100;

let plane = rootEntity.createChild('sphere');
let planeRenderer = sphere.addComponent(Mesh);
planeRenderer.mesh = PrimitiveMesh.createPlane(engine);
planeRenderer.material = mtl;
```

- [Cylinder](${book.api}classes/core.primitivemesh.html#createcylinder) **圆柱**

```typescript
let cylinder = rootEntity.createChild('cylinder');
let cylinderRenderer = cylinder.addComponent(MeshRenderer);
cylinderRenderer.mesh = PrimitiveMesh.createCylinder(engine, 2, 3, 5, 32);

// 创建材质
let mtl = new BlinnPhongMaterial(engine, 'test_mtl2', false);
mtl.ambient = new Vector4(0.75, 0.25, 0.25, 1);
mtl.shininess = 100;
cylinderRenderer.material = mtl;
```
