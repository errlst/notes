定义三角形时，根据顶点的定义顺序，可以将三角形分为顺时针三角形和逆时针三角形。

opengl 在渲染图元时，就是根据该环绕顺序判断三角形是正向三角形还是背向三角形。

opengl 默认将逆时针旋转的面视为正向面，也可以通过 `glFrontFace(GL_CCW | GL_CW)` 设置正向面。

> `GL_CW` 表示顺时针旋转为正向面。

`glEnable(CL_CULL_FACE)` 开启面剔除。

`glCullFace(GL_FRONT | GL_BACK | GL_FRONT_AND_BACK)` 设置需要剔除的面。
