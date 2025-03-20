一个摄像机由以下几个向量构成：世界空间中的坐标、观察的方向、指向相机右侧的向量和指向相机上方的向量。即一个以相机为原点的坐标系。

# 相机方向

按照惯例，相机右侧作为 +x 轴、相机上方作为 +y 轴，相机看向的方向作为 -z 轴。

获取指向摄像机 +z 轴的方向向量：

```cpp
auto camera_pos = QVector3D{1, 1, 1};
auto camera_target = QVector3D{0, 0, 0};
auto camera_direction = (camera_pos - camera_target).normalized();
```

# 上轴和右轴

通常直接定义 +y 轴，然后通过向量叉乘计算 +x 轴。

```cpp
auto camera_up = QVector3D{0, 1, 0};
auto camera_right = QVector3D::crossProduct(camera_up, camera_direct);
```

# LookAt

LookAt 矩阵可以将世界坐标转换到已经定义好的观察坐标中。LookAt 矩阵会创建一个看向目标的观察矩阵。

$$
LookAt

=

\left[
\begin{array}{}
R_x & R_y & R_z & 0 \\
U_x & U_y & U_z & 0 \\
D_x & D_y & D_z & 0 \\
0 & 0 & 0 & 1
\end{array}
\right]

*

\left[
\begin{array}{}
1 & 0 & 0 & -P_x \\
0 & 1 & 0 & -p_y \\
0 & 0 & 1 & -P_z \\
0 & 0 & 0 & 1
\end{array}
\right]
$$

其中 $R$、$U$、$D$ 分别是右向量、上向量和方向向量，$P$ 是相机位置向量。

# 简单相机类

```cpp
#pragma once

#include <glm/ext.hpp>
#include <glm/glm.hpp>

struct camera_t {
    camera_t() { update(); }

    auto view_matrix() -> glm::mat4 {
        return glm::lookAt(position_, position_ + front_, up_);
    }

    auto move(float front, float right, float delta_time) {
        float distance = speed_ * delta_time;
        position_ += front_ * front * distance;
        position_ += right_ * right * distance;
    }

    auto rotate(float x_offset, float y_offset) {
        x_offset *= sensitivity_;
        y_offset *= sensitivity_;

        yaw_ += x_offset;
        pitch_ = std::max(-89.f, std::min(pitch_ + y_offset, 89.f));
        update();
    }

    auto zoom(float off) {
        zoom_ = std::max(1.f, std::min(zoom_ - off, 45.f));
    }

    auto update() -> void {
        front_.x = glm::cos(glm::radians(yaw_)) * glm::cos(glm::radians(pitch_));
        front_.y = glm::sin(glm::radians(pitch_));
        front_.z = glm::sin(glm::radians(yaw_)) * glm::cos(glm::radians(pitch_));
        front_ = glm::normalize(front_);
        right_ = glm::normalize(glm::cross(front_, glm::vec3{0, 1, 0}));
        up_ = glm::normalize(glm::cross(right_, front_));
    }

    glm::vec3 position_{0, 0, 0}; // 位置
    glm::vec3 front_{0, 0, -1};   // 前向量
    glm::vec3 up_{0, 1, 0};       // 上向量
    glm::vec3 right_{};           // 右向量

    float yaw_{-90.0f}; // 偏航角
    float pitch_{0.0f}; // 俯仰角

    float speed_{2.5f};       // 移动速度
    float sensitivity_{0.1f}; // 旋转零敏度
    float zoom_{45.0f};       // 缩放值
};
```