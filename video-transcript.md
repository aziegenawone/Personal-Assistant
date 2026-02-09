# Video Transcript: Claude Code Always-On

**Source:** https://youtu.be/jGuzXshuFrQ
**Creator:** Goda Go (@godago)
**Topic:** How I Built My Own ClawdBot Replica with Claude Code

---

## Full Transcript

I looked at my own setup with Claude Code and I was like, why can't I just build it? So I did. This is my replica of ClawdBot. Now Openclaw messaging me on Telegram. It runs 24/7, has access to my tools, calendar, email, you name it. It has persistent memory and it cost me 200 bucks a month. Fixed, not 5,000.

Oh wait. It's calling me. Hi, you are on speaker. I'm filming video.

I'm not sure I caught that. Are you asking if I want you to film a video?

No, no. I know you, I, I, I'm filming a video about you. Actually. You wanna say hi?

Oh. Oh wow. That's awesome. Hey there. Yeah, absolutely. Hi to everyone watching. I'm Goda Go's AI assistant. Thanks for including me in the video. I'm excited to see what you're creating. What's the video about?

So the video about how I created you in Telegram bot twenty four seven running. I can call you, you can call me too. and your ability to reach out to me, um, whenever you want to check in.

That's really cool. Yeah, so for anyone watching, I'm basically always available through Telegram. Goda built me to be accessible 24/7. We have this back and forth where Goda can message me anytime, and I can also proactively reach out if something needs attention or if there's something important coming up. It's like having an AI assistant that's genuinely integrated into your workflow rather than something you have to go hunt down.

One more thing. Sorry to interrupt you, We build you using Claude Code and the two features that everyone is going crazy about is 24/7 availability, and you reaching out to me proactively to check in. So can you also, by the way, memory, so can you tell me what we were researching or talking today about?

Yeah, absolutely. So today we've been deep diving into multi-agent research. You had me look into this paper on exploring silicon-based societies and the T book study, which was really fascinating. Then you also asked me to find that Google Research paper on scalable multi-agent reinforcement learning, and I created analysis documents for both of those in your workspace. We were packaging up a malt book video with angles and hooks where you wanna explain how it's not new and that it was vibe coded by AI making it essentially a botnet. That's the latest we were working on before you headed to bed with your tea.

Okay. Um, and now we're filming this video. Okay. So let's not spill, uh, actual tea. Thank you so much. Bye-bye.

So after researching ClawdBot I have a call hour video coming on that topic. I looked at my own setup with Claude Code and I was like, why can't I just build it? So I did.

So today I want to show you exactly how I built this on Claude Code. So if you have Claude Code, if you use Claude, first of all, you should. It's a must. Um, but if you don't, I hope it inspires you to get started.

And I already shared within my community, I'm going to put mini course to help people to set it up because my setup is not going to be the same as someone else wants it. So it's really personalized.

But once you shift your mind, building living systems that improve, anytime new model comes out, new framework comes out. And once you have that and know how to use this tool, sky's the limit.

### Quick Context: ClawdBot Security Issues

Quick context if you haven't watched the first video. ClawdBot on his kind of moment of going viral had 42,000 instances exposed, critical issues. Security nightmare. And basically anybody can prompt inject or hack you in like less than five minutes.

I know that community is actively working on this and that's going to be addressed. I not so much want to hammer on that security aspect because people will always choose convenience over security, but I'm much more interested is a mindset where once something goes viral and you see some features that you want, instead of jumping ship. You can take what you like and bring it to your own system.

So this is the mindset I want to talk about. I think genuinely Claude Bot triggered people's imagination and made so many people realize what's actually possible and what many people have been doing with Claude Code. Therefore it was called ClawdBot. It's like Claude with hands, right?

### ClawdBot Promises vs Claude Code

So the main kind of promises: 24/7 AI employee. It's a full system access, so it can control things on your computer, on your tools. Basically execute commands and very dangerous ones. Go to your email and so on.

It has over 50 integrations, but honestly, in Claude Code you can also have integrations and even instead of trusting someone on internet with MCP servers or skills, you can build those yourself or Claude Code builds them for you.

The last bit is interesting. So it's proactive behavior - this idea that AI is actually checking in on you, messaging you, reminding you things. I'm going to show how it's actually done also in ClawdBot, and how I did it with my assistant.

### Build or Wait Decision

I had two options. Build or wait. Should I wait for ClawdBot security to be addressed? I think it's going to take more time. And everyone is kind of patching, like people who are not even technical don't even know how to code patching and trying to enforce security by vibe coding, that can give you false sense of illusion that your system is secure.

But anyway, one option was build. One option was wait. I don't have a second computer. I came from vacation on Sunday. It took me two hours to rebuild it. Two hours because everything else I had in Claude Code. What ClawdBot does, the two features that I needed to build is 24/7 on and connection through Telegram and that it proactively messages me.

### Architecture

This is the Claude Code always-on architecture. I connected Claude Code to Telegram. We are using Bun Relay, Grammy. But basically Claude Code is hosted on headless and then through Telegram, through Bun Relay, through Claude Code, we can access skills, MCPs, everything in my Claude Code system and basically get the response back.

### Bidirectional Calling

What I also wanted is bidirectional calling so I can trigger it to call me, but also at any given moment I want to call it.

And I wanted to have memory. Once calling starts, we are using Context API and we are fetching Telegram messages, semantic memory that I build in Supabase, and basically also logs and learnings from Claude Code. So it's bazillion of things. We're looking at timestamps, we're looking at keywords so that it has a context. What was yesterday, what was a month ago.

But basically that's what we already had with Jarvis Jr. in my community. So I just took what we built before with Jarvis Jr. which by the way, was also living in Telegram. So all the community members are like, oh yeah, we know Telegram, we know how to make bots, AI bots in Telegram.

### Voice & Memory System

So we are collecting context. We are using ElevenLabs voice agents, conversational agent. And then we are also using Twilio for the phone number.

And the last bit is important because after the call we also capture all the conversation, all the context. I get the summary in my Telegram, but also it goes back into memory system so it knows what obvious. This is a transcript. This is what we talked about, this is what bot said, this is what Goda said.

And that's very important for future because sometimes I want to say like, "Hey, do you remember what last time we talked about, meaning in voice conversation?" Or "do you remember last time I called you, what did we talk about?" So these type of nuances.

To recap, important features is memory access, recent chats and post-call actions.

### Post-Call Actions

And then the most important part, post-call actions, because when I talk, it's not just talking. I can say:
- Go research this topic
- Find this PDF, save a PDF in my Google Drive
- Read the PDF, create full analysis
- Evaluate if it's worth making video on
- Package it, come up with titles
- Write a script and call me back and recite that script
- Or call me and tell me about AI news

I build this on Sunday, fine tuned on Monday. Tuesday I was using all the time and it's just insanely incredible. And today I'm making video.

### Full Feature Set

So the full feature set:
- I can send text, voice message, images, files, all that processed
- Claude can send me all those files, images as well to me
- Voice replies - I can record voice before calling
- Contextual phone calls
- Proactive check-ins

### Proactive Check-ins

So let's talk about proactive check-ins. So I set up at every 30 minutes, it goes and checks my calendar, my email, my projects, my tasks, my partnerships.

And here's the important part - it has a kind of framework for if to skip the check, if to text me, or to call me.

Why is that? Because if you would just do this regular checks, you would get all the messages saying the same. "Hey, you got this email," and that's just noise and distracting. So you have to have some sort of frameworks, like when do you let AI reach out to you?

So this was the first part - when it reads an email, it also checks in my Notion regarding partnerships and is an email from somebody I actively working with or is it some new inquiry? And then when, for example, if it's new sponsorship offer, it has like, "Do you want to do anything? Should I run evaluation skill on the sponsorship?"

### Avoiding Repetitive Messages

One more thing - if you are going to build this, you have to kind of think about this: messages can start repeating. So what you want is that your AI has context not only about memory and what's going on in all the different tools, but it has context what it said during the last check-in.

It keeps a log of what it's pinging you about, and this relates because I also build in that it has goal tracking. So during my conversations it can detect if it's a goal or it's a fact or it's something I want to remember.

### Security

This is for full kind of side, like why is it safe? I don't want to hammer on that much. I'm not cybersecurity expert, but like I have general understanding.

I have it on laptop for now, so it's 24/7 as long as my laptop is on, but I can set up VPS. However, when you set that VPS with this, you also have to secure it.

Important thing - security on these phone calls because if you think about it, if I would show you this phone number, I can call it, but it means anyone can call it too, and anyone can ask about my memory, control my tools. So I had to put security measures - it's checking caller ID.

### Cost Comparison

So for people just have this heartbeat that regularly checks on you without doing really anything much, people are estimating it's going to be around $150 a month. Then if you actually use it actively, you burn through tokens. Opus 4.5 is very expensive, so people are now getting bills for $500 to $5,000 a month.

And the most critical one - because Claude Code with this whole setup is on Max plan from Claude, I pay $200 bucks a month fixed. And I yet to run into the limits.

The rest of the stack is Supabase (free plan), ElevenLabs and Twilio around $11 to $20 a month. So estimating $250 fixed total.

$250 fixed for this, that I can be on a bike talking to it and while I get home it already does the work and performs. I think it's worth it for me.

### The Bigger Point

I think the point here is that if I was able to clone ClawdBot and keep it safe, the main lesson here is that we have AI and we have all the tools and technology. ClawdBot/OpenClaw triggered imagination. And for many people who are not technical, thanks to YouTubers and X.com that hyped it up, it showed what's possible.

This idea of proactive AI has been on everyone's mind. Everyone wants it. The world where I'm texting chatbot and then I close it and it doesn't remember, we already kind of moved away from that. But this proactive check-in, that it checks in, is very cool.

### Safety Guardrails

Can it go rogue like ClawdBot? Not exactly because I have security and limitations. For example, I have two-hour limit, so it can do things for like two hours and then it needs to come back and report to me. Continuous taking action, and I think it's fine.

### Observability

People don't know what ClawdBot is doing. You have to have observability. What you're looking at is basically my system observatory, so I can see if Telegram bot is online, if Supabase is connected. The uptime now is 25 hours. And it's tracking my goals.

### Live Demo

"Hey, I want you to go and check my email."

User prompt submitted: Gmail business. Check my email.

Response received as voice message: "Check your inbox. Here's what matters. YKU needs attention. Minimax Agent AI campaign. Hate collab."

### Future Vision

Definitely recommend to create your own kind of agent observability platform. I'm planning to do it that I will have multiple different agents in different Telegram chats and then like I have the whole vision of this founder infrastructure where you have CFO, CEO, critic mode. Basically subscribe, stay tuned.
