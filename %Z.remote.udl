/// Handle remote system managent requests
Class %Z.remote Extends %Persistent [ Final ]
{

Parameter SLEEP As %Integer = 5;

Property Server As %String;

Property IPaddr As %String(PATTERN = "1.3N1"".""1.3N1"".""1.3N1"".""1.3N");

Property Listener As %Integer [ InitialExpression = 59999 ];

Property Connection As %String(COLLATION = "EXACT") [ Calculated, SqlComputeCode = { set {*}={IPaddr}_":"_{Listener}}, SqlComputed ];

Index con On Connection [ Unique ];

/// hang around and wait for request
ClassMethod Server() As %Status
{
    set local=..%New()
    quit local.loop()
}

Method loop() As %Status
{
    set tcp="|TCP|"_..Listener
    set $ztrap="recover"
    set $ze="READ"
recover
    close tcp
    if $ze["READ"    ; restart connection
    else  quit $test ; terminate fo any other error 
    open tcp:(/PORT=..Listener:/PSTE=1:/SEN=1:/CON=999):1 else  quit $test
    for  { 
        use tcp read request:..#SLEEP if $l(request)  do ..dispatch(.tcp,request)
    }
    quit $$$OK     ;; will never happen
}

/// just a simple example how this may work   
Method dispatch(ByRef tcp, request As %String) [ ProcedureBlock = 0 ]
{
     set action=+request   
     if $L($TEXT(@action)) do @action
     else  set reply="not implemented"
reply
     use tcp write reply,!,*-2 
     quit
9 ;STOP
    set reply="Server stopped" do reply ; final reply migth is lost
    hang ..#SLEEP ZTRAP "STOP" 
    quit         

1 ;INST
    set reply="InstanceName: "_^SYS("GREDIR","NSACT")
    quit 

2 ;PRCC
    set pid=0
    &SQL(Select count(pid) into :pid from   %SYS.ProcessQuery)
    set reply="actual process count: "_pid
    quit
}

/// client connects to remote server to get some info
ClassMethod Client() As %Status
{
 #dim srv as %Z.remote = ""
    set sql="SELECT * from %Z.remote"
    set stmt=##class(%SQL.Statement).%New()
    set sc=stmt.%Prepare(sql)
       ,res=stmt.%Execute()
    do res.%Display()
    do {
        read !,"select ServerID:",sid,!
        if 'sid return '$$$OK
        set srv=##class(%Z.remote).%OpenId(sid)
    } while '$ISOBJECT(srv)
    set tcp="|TCP|"_srv.Listener
    set $ZE="READ"
    set $ZTRAP="reset"  
reset    
   close tcp
   if $ZE'["READ",$ZE'["WRITE"  set $ZT="" return '$$$OK 
   open tcp:(srv.IPaddr:/PORT=srv.Listener):1  else  set $ZT="" return '$$$OK 
   use 0
   set (reply,cmd)="" 
   while '$L(cmd) {
     read !,"1=INSTANCE NAME",
          !,"2=PROCESS COUNT",
          !,"9=STOP REMOTE PING SERVER",
          !,"*=EXIT",
          !,": ",cmd,!
     if "*129"'[cmd set cmd=""
     if cmd="*" ZTRAP "STOP"
   }
   use tcp Write cmd,!,*-2
   use 0 write "send: ",cmd,!
   use tcp 
   for i=1:1:10 read reply:..#SLEEP  if $L(reply) quit
   else  set reply= "??? missing ??? "
   use 0 write reply,! 
   goto reset
}
}
