<div align="center">

## Simple POP3 Console MailChecker C\# class


</div>

### Description

This example below is a very simple class that connect to you're isp provider and looks for mail.

This class shows how to use the TCPClient class,

how to create a new Thread , how to capture events (AsyncCallBack) from recieving data on the socket.
 
### More Info
 


<span>             |<span>
---                |---
**Submitted On**   |
**By**             |[Peter V\.](https://github.com/Planet-Source-Code/PSCIndex/blob/master/ByAuthor/peter-v.md)
**Level**          |Beginner
**User Rating**    |4.4 (31 globes from 7 users)
**Compatibility**  |C\#
**Category**       |[Internet/ Browsers/ HTML](https://github.com/Planet-Source-Code/PSCIndex/blob/master/ByCategory/internet-browsers-html__10-9.md)
**World**          |[\.Net \(C\#, VB\.net\)](https://github.com/Planet-Source-Code/PSCIndex/blob/master/ByWorld/net-c-vb-net.md)
**Archive File**   |[](https://github.com/Planet-Source-Code/peter-v-simple-pop3-console-mailchecker-c-class__10-115/archive/master.zip)





### Source Code

<TABLE BORDER=1 CELLSPACING=0 CELLPADDING=2 BGCOLOR=#FFFFFF>
<TR><TD>
<TT><PRE>using System;
using System.Net.Sockets;
using System.Text;
using System.IO;
using System.Threading;
namespace myMail
{
	/// &lt;summary&gt;
	/// POP mail Retrieve
	/// &lt;/summary&gt;
	public class clsPOP
	{
		private TcpClient myPOP;
		private NetworkStream networkStream;
		private byte[] bytes;
		private string mTxtPassword;
		private string mTxtUsername;
		private string mTxtPopServer;
		private ConnState ConnectionState;
		private bool blnClosePop;
		private bool blnExitPop;
		private enum ConnState
		{
			Connect = 0,
			User  = 1,
			Passw  = 2,
			Stat  = 3,
			List  = 4,
			Mail  = 5,
			Quit  = 6,
		}
		public clsPOP()
		{
			myPOP = new TcpClient();
			mTxtPopServer = &quot;Unknown&quot;;
			mTxtPassword = &quot;Unknown&quot;;
			mTxtUsername = &quot;Unknown&quot;;
			blnExitPop = false;
			bytes = new byte[myPOP.ReceiveBufferSize];
			Thread.AllocateDataSlot();
			Console.WriteLine(&quot;P O P 3 M a i l C h e c k  V 1 . 0 &quot;);
			Console.WriteLine(&quot;______________________________________&quot;);
		}
		public string Password
		{
			set
			{
				mTxtPassword = value;
			}
			get
			{
				return mTxtPassword;
			}
		}
		public string UserName
		{
			set
			{
				mTxtUsername = value;
			}
			get
			{
				return mTxtUsername;
			}
		}
		public string PopServer
		{
			set
			{
				mTxtPopServer = value;
			}
			get
			{
				return mTxtPopServer;
			}
		}
		private string GetTCPData()
		{
			int NumBytesRecieved;
			try
			{
				NumBytesRecieved =
					networkStream.Read(bytes, 0, (int) myPOP.ReceiveBufferSize);
				return (Encoding.ASCII.GetString(bytes).Substring(0,NumBytesRecieved));
			}
			catch (Exception e )
			{
				return(e.ToString());
			}
		}
		// Event when data is recieved from server
		public void TcpDataRecieve( IAsyncResult ar )
		{
			int BeginPos;
			int EndPos;
			if (ConnectionState != ConnState.Quit)
			{
				Console.WriteLine(&quot;{0}&quot; , GetTCPData());
			}
			else
			{
				BeginPos = String.Compare(GetTCPData(),&quot;Subject&quot;);
			}
			MailChecking();
			networkStream.Flush();
			networkStream.BeginRead(bytes,1,1,new AsyncCallback( TcpDataRecieve ), myPOP);
		}
		private void MailChecking()
		{
			string Ret;
			switch(ConnectionState)
			{
				case ConnState.Connect :
					Ret = &quot;USER &quot; + mTxtUsername + <B>'\n'; </B>
					networkStream.Write(System.Text.Encoding.ASCII.GetBytes(Ret),0, Ret.Length);
					ConnectionState = ConnState.User;
					break;
				case ConnState.User :
					Ret = &quot;PASS &quot; + mTxtPassword + <B>'\n'; </B>
					networkStream.Write(System.Text.Encoding.ASCII.GetBytes(Ret),0, Ret.Length);
					ConnectionState = ConnState.Passw;
					break;
				case ConnState.Passw :
					Ret = &quot;STAT &quot; + <B>'\n'; </B>
					networkStream.Write(System.Text.Encoding.ASCII.GetBytes(Ret),0, Ret.Length);
					ConnectionState = ConnState.Stat;
					break;
				case ConnState.Stat :
					Ret = &quot;LIST &quot; + <B>'\n'; </B>
					networkStream.Write(System.Text.Encoding.ASCII.GetBytes(Ret),0, Ret.Length);
					ConnectionState = ConnState.Mail;
					break;
				case ConnState.Mail :
					Ret = &quot;TOP 1 1 &quot; + <B>'\n'; </B>
					networkStream.Write(System.Text.Encoding.ASCII.GetBytes(Ret),0, Ret.Length);
					ConnectionState = ConnState.List;
					break;
				case ConnState.List :
					Ret = &quot;QUIT &quot; + <B>'\n'; </B>
					networkStream.Write(System.Text.Encoding.ASCII.GetBytes(Ret),0, Ret.Length);
					ConnectionState = ConnState.Quit;
					break;
				case ConnState.Quit :
					myPOP.Close();
					blnClosePop = true;
					break;
			}
		}
		public void Connect()
		{
			try
			{
				blnClosePop = false;
				Console.WriteLine(&quot;.Connecting to : {0}&quot;, mTxtPopServer);
				myPOP.Connect(PopServer,110);
				ConnectionState = ConnState.Connect;
				networkStream = myPOP.GetStream();
				networkStream.BeginRead(bytes,1,1,new AsyncCallback( TcpDataRecieve ), myPOP);
				while (!blnClosePop &amp;&amp; !blnExitPop)
				{
					Thread.Sleep(100);
				}
			}
			catch (Exception e )
			{
				Console.WriteLine (&quot;Error Connecting {0}&quot; , e.ToString());
				myPOP.Close();
			}
		}
		public void Close()
		{
			myPOP.Close();
			blnExitPop = true;
		}
		public void ThreadHandler()
		{
			Thread currentThread = Thread.CurrentThread;
			this.Connect();
		}
	}
}</PRE></TT>
</TD></TR>
</TABLE>
<H4>Main Class , that use this class above</H4><BR>
<H5>
static void Main(string[] args)<BR>
{<BR>
clsPop myPop = new clsPOP();<BR>
myPop.UserName = "your username";<BR>
myPop.Password = "your password";<BR>
myPop.PopServer = "your isp pop server";<BR>
ThreadStart PopMail = new ThreadStart(myPop.ThreadHandler);<BR>
myPopMail = new Thread(PopMail);<BR>
myPopMail.Start();<BR>
while(myPopMail.IsAlive)<BR>
 {<BR>
 Thread.Sleep(100);<BR>
 }<BR>
}<BR></H5>

