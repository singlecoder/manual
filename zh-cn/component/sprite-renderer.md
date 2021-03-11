# 图片显示

[SpriteRenderer](${book.api}classes/core.spriterenderer.html) 组件用于在 3D/2D 场景中显示图片。

## 基本使用

1、下载图片纹理([Texture](${book.manual}resource/texture.md))下载方法请参考[资源加载](${book.manual}resource/resource-manager.md)    
2、通过 texture 创建 [Sprite](${book.api}classes/core.2d.sprite.html) 对象    
3、创建 [SpriteRenderer](${book.api}classes/core.2d.spriterenderer.html) 组件显示图片

```typescript
import {
  AssetType,
  Camera,
  Script,
  Sprite,
  SpriteRenderer,
  SystemInfo,
  Texture2D,
  Vector3,
  WebGLEngine
} from "oasis-engine";

engine.resourceManager
  .load<Texture2D>({
    url: "https://gw.alipayobjects.com/mdn/rms_7c464e/afts/img/A*d3N9RYpcKncAAAAAAAAAAAAAARQnAQ",
    type: AssetType.Texture2D
  })
  .then((texture) => {
    const spriteEntity = rootEntity.createChild(`sprite`);
    // 给实体添加 SpriteRenderer 组件
    const spriteRenderer = spriteEntity.addComponent(SpriteRenderer);
    // 通过 texture 创建 sprite 对象
    const sprite = new Sprite(engine, texture);
    // 设置 sprite
    spriteRenderer.sprite = sprite;
  });
```

