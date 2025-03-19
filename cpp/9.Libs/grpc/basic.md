## 基本概念

- service。service 是 rpc 的集合，protobuf 编译器会生成服务端和客户端需要的代码。

- stub。stub 是客户端的代理对象，每个 service 都有对应的 stub，用于 rpc 调用。

- channel。channdel 是客户端与服务器间的抽象连接。

- context。context 负责传递 rpc 元数据（如超时、认证）和流量控制。

- metadata。metadata 是以键值对形式传递的附加信息，通过 context 获取。

- status。status 表示 rpc 调用结果状态。

## echo 示例

<div>
echo.proto

```proto
syntax = "proto3";

package echo;

/* 定义一个 service */
service EchoService {

  /* 定义一个 rpc */
  rpc echo (EchoRequest) returns (EchoResponse) {}
}

message EchoRequest {
  string message = 1;
}

message EchoResponse {
  string message = 1;
}
```

</div>

<div>
server.cpp

```cpp
#include "echo.grpc.pb.h"
#include <grpcpp/grpcpp.h>
#include <print>
#include <ranges>

class EchoServiceImpl final : public echo::EchoService::Service {
  auto echo(grpc::ServerContext *ctx, const echo::EchoRequest *request,
            echo::EchoResponse *replay) -> grpc::Status override {
    replay->set_message(
        request->message() |
        std::ranges::views::transform([](char c) { return std::toupper(c); }) |
        std::ranges::to<std::string>());
    return grpc::Status::OK;
  }
};

auto main() -> int {
  auto echo_service = EchoServiceImpl{};
  auto builder = grpc::ServerBuilder{};
  builder.AddListeningPort("0.0.0.0:8888", grpc::InsecureServerCredentials());
  builder.RegisterService(&echo_service);

  auto server = builder.BuildAndStart();
  server->Wait();
  return 0;
}
```

</div>

<div>
client.cpp

```cpp
#include "echo.grpc.pb.h"
#include <grpcpp/grpcpp.h>
#include <print>

class EchoClient {
public:
  EchoClient(std::shared_ptr<grpc::Channel> channel)
      : stub_(echo::EchoService::NewStub(channel)) {}

  auto echo(const std::string &msg) -> std::string {
    auto request = echo::EchoRequest{};
    request.set_message(msg);

    auto response = echo::EchoResponse{};
    auto ctx = grpc::ClientContext{};

    auto status = stub_->echo(&ctx, request, &response);
    return status.ok() ? response.message() : "RPC failed";
  }

private:
  std::unique_ptr<echo::EchoService::Stub> stub_;
};

auto main() -> int {
  auto client = EchoClient{grpc::CreateChannel(
      "localhost:8888", grpc::InsecureChannelCredentials())};
  while (true) {
    auto msg = std::string{};
    std::cin >> msg;
    std::println("{}", client.echo(msg));
  }

  return 0;
}
```

</div>

<div>
xmake.lua

```lua
add_rules("mode.debug", "mode.release")

set_languages("c++23")
add_cxxflags("-Wall")
set_toolchains("gcc-14")

add_requires("protobuf-cpp", {system=false})
add_requires("grpc", {system=false})
add_packages("protobuf-cpp", "grpc")

-- 自动编译 proto 文件，输出到 build/.gen 目录下
add_rules("protobuf.cpp")
add_files("proto/*.proto", {proto_rootdir="proto", proto_grpc_cpp_plugin=true})

target("server")
    set_kind("binary")
    add_files("src/server.cpp")

target("client")
    set_kind("binary")
    add_files("src/client.cpp")
```

</div>
