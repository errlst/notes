`QOpenGLWidget` 提供在 qt 程序中集成 opengl 的功能。

`QOpenGLWidget`提供了三个虚函数，可以在其中执行典型的 gl 任务：

- `initializeGL()`，初始化 gl。

- `paintGL()`，渲染 gl，当需要更新时调用。

- `resizeGL()`，当窗口大小改变、第一次显示时调用，此时通常设置 glviewport、glprojection 等。

> 在这三个函数调用时，自动获取 opengl 上下文。如果需要在其他地方调用 gl 相关函数，需要通过 `makeCurrent()` 获取上下文，并使用 `doneCurrent()` 释放上下文。

## 示例

```cpp
// glwidget.h
#pragma once

#include <QOpenGLExtraFunctions>
#include <QOpenGLWidget>

class GLWidget : public QOpenGLWidget, protected QOpenGLExtraFunctions {
  Q_OBJECT;

public:
  GLWidget(QWidget *parent = nullptr) : QOpenGLWidget(parent) {}

  ~GLWidget() {}

protected:
  auto initializeGL() -> void override;

  auto paintGL() -> void override;

  auto resizeGL(int w, int h) -> void override;
};

// glwidget.cc
#include "glwidget.h"

auto GLWidget::initializeGL() -> void {
  initializeOpenGLFunctions();
  glClearColor(0, 0, 0, 0);
  resizeGL(width(), height());
}

auto GLWidget::paintGL() -> void {
  glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT);
  glColor3f(1, 1, 1);

  glBegin(GL_TRIANGLES);
  glVertex3f(-.5f, -.5f, 0);
  glVertex3f(.5f, -.5f, 0);
  glVertex3f(0, .5f, 0);
  glEnd();
}

auto GLWidget::resizeGL(int w, int h) -> void {
  auto side = std::min(w, h);
  glViewport((w - side) / 2, (h - side) / 2, side, side);
}

```
