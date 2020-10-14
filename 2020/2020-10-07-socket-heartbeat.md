
```java
    public void run() {
        ServerSocket serverSocket = null;
        try {
            boxService = BeanContext.getApplicationContext().getBean(BoxService.class);
            serverSocket = new ServerSocket(port);
            TboxContext tboxContext = TboxContext.getInstance();

            while (true) {
                Socket socket = serverSocket.accept();
                logger.info("Rcv Connection established : " + socket.getRemoteSocketAddress());
                tboxContext.setOnline(true);
                tboxContext.setTboxRemoteAddress(socket.getRemoteSocketAddress().toString());
                tboxContext.setSocket(socket);

                BufferedReader bufferedReader = new BufferedReader(new InputStreamReader(
                        socket.getInputStream(), StandardCharsets.UTF_8));

                char[] buffer = new char[BUFFER_SIZE];

                while (bufferedReader.read(buffer) != -1) {
                    JSONObject jsonObject = JSONObject.parseObject(String.copyValueOf(buffer));
                    //json对象转Map
                    Map<String,Object> map = (Map<String,Object>)jsonObject;
                    Box box = new Box();
                    box.setContentType((String) map.get("contentType"));
                    box.setValue(jsonObject.toJSONString());
                    boxService.insert(box);

                    Arrays.fill(buffer, '\0');
                    if (!isConnected(socket)) {
                        TboxContext.getInstance().setOnline(false);
                        break;
                    }
                }
                logger.info("Rcv Connection closed : " + socket.getRemoteSocketAddress());
                socket.close();
                bufferedReader.close();
                tboxContext.setSocket(null);
                tboxContext.setTboxRemoteAddress("");
                tboxContext.setOnline(false);
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
```