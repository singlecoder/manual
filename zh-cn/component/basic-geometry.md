# 基础几何体

常用几何体统一在 [PrimitiveMesh](${book.api}classes/core.PrimitiveMesh.html) 中提供，目前提供的几何体如下：

- [createCuboid](${book.api}classes/core.cuboidgeometry.html) **立方体**

```typescript
const entity = rootEntity.createChild('cuboid');
entity.transform.setPosition(0, 1, 0);
const renderer = entity.addComponent(MeshRenderer);
renderer.mesh = PrimitiveMesh.createCuboid(engine);
// Create material
const material = new BlinnPhongMaterial(engine);
material.emissiveColor.setValue(1, 1, 1, 1);
renderer.setMaterial(material);
```

- [createSphere](${book.api}classes/core.spheregeometry.html) **球体**

```typescript
const entity = rootEntity.createChild('sphere');
entity.transform.setPosition(0, 1, 0);
const renderer = entity.addComponent(MeshRenderer);
renderer.mesh = PrimitiveMesh.createSphere(engine);
// Create material
const material = new BlinnPhongMaterial(engine);
material.emissiveColor.setValue(1, 1, 1, 1);
renderer.setMaterial(material);
```

- [createPlane](${book.api}classes/core.spheregeometry.html) **平面**

```typescript
const entity = rootEntity.createChild('plane');
entity.transform.setPosition(0, 1, 0);
const renderer = entity.addComponent(MeshRenderer);
renderer.mesh = PrimitiveMesh.createPlane(engine);
// Create material
const material = new BlinnPhongMaterial(engine);
material.emissiveColor.setValue(1, 1, 1, 1);
renderer.setMaterial(material);
```

- [createCylinder](${book.api}classes/core.circlegeometry.html) **圆柱**

```typescript
const entity = rootEntity.createChild('cylinder');
entity.transform.setPosition(0, 1, 0);
const renderer = entity.addComponent(MeshRenderer);
renderer.mesh = PrimitiveMesh.createCylinder(engine);
// Create material
const material = new BlinnPhongMaterial(engine);
material.emissiveColor.setValue(1, 1, 1, 1);
renderer.setMaterial(material);
```
