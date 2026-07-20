[phones]

exten => *100,1,Answer()
same => n,VoiceMailMain(100@default)
same => n,Hangup()

exten => *200,1,Answer()
same => n,VoiceMailMain(200@default)
same => n,Hangup()

exten => 100,1,NoOp(------- CALLING JAMES DESK ------)
same => n,Dial(SIP/james,5)
same => n,GotoIf($["${DIALSTATUS}" = "BUSY"]?100-busy)
same => n,VoiceMail(${EXTEN}@default,u)
same => n,Hangup()

same => n(100-busy,VoiceMail(${EXTEN}@default,b)
same => n,Hangup()


exten => 200,1,NoOp(--- Calling Mat Desk ---)
same => n,Dial(SIP/mat,5)
same => n,VoiceMail(${EXTEN}@default,u)
same => n,Hangup()

exten => *500,1,Answer()
same => n,Wait(60)
same => n,Hangup()

exten => _0X.,1,NoOp(${EXTEN:-3})
same => n,Goto(outgoing,${EXTEN:1},1)
