import os import sys import requests from telethon import TelegramClient from telethon.tl.functions.channels import CreateChannelRequest, TogglePreHistoryHiddenRequest from telethon.tl.functions.messages import ExportChatInviteRequest from telethon.errors import FloodWaitError, SessionPasswordNeededError import asyncio import random

=========================

Config from Environment

=========================

API_ID = int(os.getenv("TG_API_ID", "0")) API_HASH = os.getenv("TG_API_HASH", "" ) PHONE = os.getenv("TG_PHONE", "") TWO_FA = os.getenv("TG_2FA", None)  # Optional 2FA password

BASE_NAME = os.getenv("TG_BASE_NAME", "Group") START_NUMBER = int(os.getenv("TG_START_NUMBER", "1")) NUM_GROUPS = max(1, min(50, int(os.getenv("TG_NUM_GROUPS", "1")))) MSG_DELAY = float(os.getenv("TG_MSG_DELAY", "1")) GROUP_DELAY = float(os.getenv("TG_GROUP_DELAY", "5"))

messages = [ "🌟 Welcome to our private community!", "This is a private group for trusted members.", "Feel free to share and discuss here!", "We'll keep this space positive and respectful.", "Stay tuned for exclusive updates!", "Thank you for being part of our community!", ]

async def main(): print("=" * 50) print("CUSTOMIZABLE PRIVATE GROUP CREATOR (HOSTING MODE)") print("=" * 50)

session_name = f"session_{random.randint(1000, 9999)}"
client = TelegramClient(session_name, API_ID, API_HASH)
await client.start()

if not await client.is_user_authorized():
    print(f"Sending verification code to {PHONE}...")
    await client.send_code_request(PHONE)

    code = os.getenv("TG_CODE")  # you must set this once manually
    if not code:
        print("❌ No TG_CODE provided in env. Add it after receiving SMS.")
        sys.exit(1)

    try:
        await client.sign_in(PHONE, code)
    except SessionPasswordNeededError:
        if TWO_FA:
            await client.sign_in(password=TWO_FA)
        else:
            print("❌ 2FA enabled, but TG_2FA not set.")
            sys.exit(1)

me = await client.get_me()
print(f"\n✓ Logged in as: {me.first_name}")
print("=" * 50)

successful_groups = 0
group_links = []

for i in range(NUM_GROUPS):
    group_number = START_NUMBER + i
    group_name = f"{BASE_NAME} {group_number}"

    try:
        print(f"\n--- Creating Group {i+1}/{NUM_GROUPS}: {group_name} ---")
        result = await client(CreateChannelRequest(
            title=group_name,
            about=f"Private group {group_name} for discussions",
            megagroup=True,
            broadcast=False
        ))

        channel = result.chats[0]
        print(f"✓ Created '{group_name}'")

        try:
            await client(TogglePreHistoryHiddenRequest(channel=channel, hidden=False))
            print("✓ History set to visible")
        except:
            print("Note: Could not change history settings")

        try:
            invite = await client(ExportChatInviteRequest(peer=channel))
            invite_link = invite.link
            print(f"✓ Invite link: {invite_link}")
        except Exception as e:
            print(f"Could not create invite link: {e}")
            invite_link = "No link"

        print("Sending welcome messages...")
        for msg_num, message in enumerate(messages, 1):
            try:
                await client.send_message(entity=channel, message=message)
                print(f"Message {msg_num}/{len(messages)} sent")
                await asyncio.sleep(MSG_DELAY)
            except Exception as e:
                print(f"Failed to send message {msg_num}: {e}")

        successful_groups += 1
        group_links.append(f"{group_name}: {invite_link}")

    except FloodWaitError as e:
        print(f"⏳ FloodWaitError: wait {e.seconds}s")
        break
    except Exception as e:
        print(f"❌ Failed to create {group_name}: {e}")

    if i < NUM_GROUPS - 1:
        print(f"Waiting {GROUP_DELAY}s before next group...")
        await asyncio.sleep(GROUP_DELAY)

print("\n" + "="*50)
print("CREATION SUMMARY")
print("="*50)
print(f"Base name: {BASE_NAME}")
print(f"Attempted: {NUM_GROUPS} | Successful: {successful_groups}")
if successful_groups > 0:
    for link in group_links:
        print(f"  • {link}")
else:
    print("No groups created successfully.")

await client.disconnect()
print(f"\nLogged out of {me.first_name}")
print("=" * 50)

if name == "main": asyncio.run(main())

