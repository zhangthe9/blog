---
layout: post
title: java scoket
description:
- java scoket
categories:
- java 
tags:
- socket
---
<pre class="prettyprint">
import java.io.*;
import java.net.*;

public class RemoteFileServer {
	int listenPort;

	public RemoteFileServer(int listenPort) {
		this.listenPort = listenPort;
	}


	public void acceptConnections() throws Exception {

		ServerSocket server = new ServerSocket(listenPort);
		Socket incomingConnection = null;
		while (true) {
			incomingConnection = server.accept();
			handleConnection(incomingConnection);
		}
	}
	
	public void handleConnection(Socket incomingConnection) throws Exception {

		OutputStream outputToSocket = incomingConnection.getOutputStream();
		
		InputStream inputFromSocket = incomingConnection.getInputStream();
		
		BufferedReader streamReader = new BufferedReader(new InputStreamReader(
				inputFromSocket));
		FileReader fileReader = new FileReader(
				new File(streamReader.readLine()));
		BufferedReader bufferedFileReader = new BufferedReader(fileReader);
		
		PrintWriter streamWriter = new PrintWriter(
				incomingConnection.getOutputStream());
		
		String line = null;
		while ((line = bufferedFileReader.readLine()) != null) {
			streamWriter.println(line);
		}
		
		fileReader.close();
		streamWriter.close();
		streamReader.close();

	}

	public static void main(String args[]) throws Exception {
		RemoteFileServer server = new RemoteFileServer(1001);
		server.acceptConnections();
	}
}
</pre>

<pre class="prettyprint">
import java.io.*;
import java.net.*;

public class RemoteFileClient {
	protected BufferedReader socketReader;
	protected PrintWriter socketWriter;
	protected String hostIp;
	protected int hostPort;

	// 构造方法
	public RemoteFileClient(String hostIp, int hostPort) {
		this.hostIp = hostIp;
		this.hostPort = hostPort;
	}

	// 向服务器请求文件的内容
	public String getFile(String fileNameToGet) throws Exception {
		StringBuffer fileLines = new StringBuffer();

		socketWriter.println(fileNameToGet);
		socketWriter.flush();
		String line = null;
		
		while ((line = socketReader.readLine()) != null) {
			fileLines.append(line + "\n");
		}

		return fileLines.toString();
	}

	// 连接到远程服务器
	public void start() throws Exception {
		Socket client = new Socket(hostIp, hostPort);
		
		socketReader = new BufferedReader(new InputStreamReader(
				client.getInputStream()));
		
		socketWriter = new PrintWriter(client.getOutputStream());
	}

	// 断开远程服务器
	public void close() throws Exception {
		socketWriter.close();
		socketReader.close();
	}

	public static void main(String args[]) throws Exception {
		RemoteFileClient remoteFileClient = new RemoteFileClient("127.0.0.1",
				1001);
		remoteFileClient.start();

		StringBuffer fileContents = new StringBuffer();
		fileContents.append(remoteFileClient.getFile("z:/dd.txt"));
		// remoteFileClient.close();

		System.out.println(fileContents);
	}
}
</pre>

####多线程Socket服务器
前例  一次只能处理一台客户机
handleConnection()是一个阻塞方法。只有当它完成了对当前连接的处理时，才能接受另一个客户机


<pre class="prettyprint">
import java.io.*;
import java.net.*;

public class ConnectionHandler implements Runnable {
	protected Socket socketToHandle;

	public ConnectionHandler(Socket socketToHandle) {
		this.socketToHandle = socketToHandle;
	}

	public void run() {
		try {
			PrintWriter streamWriter = new PrintWriter(
					socketToHandle.getOutputStream());
			BufferedReader streamReader = new BufferedReader(
					new InputStreamReader(socketToHandle.getInputStream()));
			String fileToRead = streamReader.readLine();
			BufferedReader fileReader = new BufferedReader(new FileReader(
					fileToRead));
			String line = null;
			while ((line = fileReader.readLine()) != null) {
				streamWriter.println(line);
			}
			fileReader.close();
			streamWriter.close();
			streamReader.close();
		} catch (Exception e) {
			System.out.println("Error handling a client: " + e);
			e.printStackTrace();
		}
	}
}
</pre>

<pre class="prettyprint">
import java.net.ServerSocket;
import java.net.Socket;

public class MultithreadedRemoteFileServer {
	int listenPort;

	public MultithreadedRemoteFileServer(int listenPort) {
		this.listenPort = listenPort;
	}


	public void acceptConnections() throws Exception{
		
			//最多5连接
			ServerSocket server = new ServerSocket(listenPort, 5);
			Socket incomingConnection = null;
			while (true) {
				incomingConnection = server.accept();
				handleConnection(incomingConnection);
			}
	}


	public void handleConnection(Socket connectionToHandle) {
		new Thread(new ConnectionHandler(connectionToHandle)).start();
	}

	public static void main(String args[]) throws Exception{
		MultithreadedRemoteFileServer server = new MultithreadedRemoteFileServer(
				1001);
		server.acceptConnections();
	}
}
</pre>

####带有连接池的Socket服务器
