cairo 是一个支持多输出的 2d 矢量图形库，包括输出到 xwindows、win32、图像等。

## 示例

```cpp
// 绘制白色背景，红色空心圆和黑色文本到图片
#include <cairo/cairo.h>

auto main() -> int {
    auto surface = cairo_image_surface_create(CAIRO_FORMAT_ARGB32, 600, 600);
    auto cr = cairo_create(surface);

    cairo_set_source_rgb(cr, 1, 1, 1);
    cairo_paint(cr);

    cairo_set_source_rgb(cr, 1, 0, 0);
    cairo_set_line_width(cr, 10);
    cairo_arc(cr, 300, 300, 200, 0, 2 * 3.1415926);
    cairo_stroke(cr);

    cairo_select_font_face(cr, "Sans", CAIRO_FONT_SLANT_NORMAL, CAIRO_FONT_WEIGHT_BOLD);
    cairo_set_font_size(cr, 40);
    cairo_move_to(cr, 200, 300);
    cairo_set_source_rgb(cr, 0, 0, 0);
    cairo_show_text(cr, "hello cairo!");

    cairo_surface_write_to_png(surface, "out.png");
    cairo_destroy(cr);
    cairo_surface_destroy(surface);

    return 0;
}
```

- `cairo_image_surface_create()`，创建指定格式的图像 surface，surface 是 cairo 绘制的场所。

- `cairo_create()`，创建绑定到 surface 的上下文。

- `cairo_set_source_rgb()`，设置上下文绘制颜色。

- `cairo_paint()`，填充整个 surface。

- `cairo_set_line_width()`，设置上下文绘制线宽。

- `cairo_arc()`，定义圆弧路径。

- `cairo_stroke()`，绘制路径到 surface。cairo 中的实际绘制操作只有 stroke，其他操作都只是定义绘制的路径。一个画图路径由多个画图 api 定义。

- `cairo_move_to()`，从指定坐标开始一个新的路径。
