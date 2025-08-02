# DisplayMyHEIC
在任何网页上正常显示你贴出的HEIF/HEIC图片。 Display your posted HEIF/HEIC images on any web page.
## 初衷/感谢开源
让我的Typecho博客可以正常显示正在推广的HEIF/HEIC格式照片。
感谢hoppergee的[heic-to](https://github.com/hoppergee/heic-to)项目！
感谢自由软件与开源社区！
## 工作原理：
1. 自动检测所有带有.heic或.HEIC扩展名的图片
2. 使用fetch API获取原始HEIC文件
3. 在浏览器中转换为JPEG格式
4. 替换图片的src属性显示转换后的图片
### 核心特性  
✅ **CSP兼容**  
不依赖 `eval()` 或 `new Function()`，符合严格的内容安全策略  
🌐 **纯前端**  
转换过程完全在浏览器中完成，无需服务器支持  
📦 **轻量级**  
仅依赖 [heic-to](https://github.com/hoppergee/heic-to) 的 CSP 安全版本  
🛠 **简单集成**  
只需添加一段脚本到您的网站  
🔄 **转换状态提示**  
- 转换中：显示加载动画  
- 转换成功：添加绿色边框  
- 转换失败：红色虚线边框 + 错误提示  
## 使用方法：
将以下代码添加到网站的`</body>`标签前，通常我们把他放在header或footer里。
在Typecho上我们可以把它放在主题文件的 header.php 中。
```javascript
<script type="module">
// 导入CSP安全版本的HEIC转换模块
import { heicTo } from 'https://cdn.jsdelivr.net/npm/heic-to@1.2.1/dist/csp/heic-to.js';

document.addEventListener('DOMContentLoaded', async function() {
    // 检查浏览器是否支持所需API
    if (!window.fetch || !window.URL || !window.Blob) {
        console.warn('浏览器不支持HEIC转换所需API');
        return;
    }
    
    // 处理HEIC图片转换
    async function processHEICImages() {
        const images = document.querySelectorAll('img[src$=".heic"], img[src$=".HEIC"]');
        if (images.length === 0) return;
        
        console.log(`找到 ${images.length} 张HEIC图片，开始转换...`);
        
        for (const img of images) {
            const src = img.src;
            const originalAlt = img.alt || '';
            const originalClass = img.className;
            
            try {
                // 添加加载状态
                img.alt = 'HEIC图片转换中...';
                img.classList.add('heic-loading');
                
                // 获取HEIC文件
                const response = await fetch(src);
                if (!response.ok) throw new Error(`HTTP错误! 状态码: ${response.status}`);
                
                const blob = await response.blob();
                
                // 转换为JPEG
                const jpegBlob = await heicTo({
                    blob: blob,
                    type: "image/jpeg",
                    quality: 0.8
                });
                
                // 创建对象URL并替换
                const jpegUrl = URL.createObjectURL(jpegBlob);
                img.onload = function() {
                    URL.revokeObjectURL(jpegUrl); // 释放内存
                    img.classList.remove('heic-loading');
                    img.classList.add('heic-converted');
                };
                img.src = jpegUrl;
                img.alt = originalAlt;
                img.className = originalClass;

            } catch (err) {
                console.error('HEIC转换失败:', err);
                img.alt = originalAlt + ' [HEIC转换失败]';
                img.classList.remove('heic-loading');
                img.classList.add('heic-error');
            }
        }
    }
    
    await processHEICImages();
});
</script>

<style>
.heic-loading {
    position: relative;
    min-height: 100px;
    background: #f5f5f5 url('data:image/svg+xml;utf8,<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 100 100"><circle cx="50" cy="50" r="40" stroke="%233498db" stroke-width="8" fill="none" stroke-dasharray="62.8 188.8"><animateTransform attributeName="transform" type="rotate" repeatCount="indefinite" dur="1s" values="0 50 50;360 50 50" keyTimes="0;1"></animateTransform></circle></svg>') no-repeat center;
    background-size: 50px;
}
.heic-converted {
    border: 2px solid #2ecc71;
}
.heic-error {
    border: 2px dashed #e74c3c;
}
```
</style>
```




