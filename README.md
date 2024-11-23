# blocking-users- #meta developer: @pipwine
from .. import loader, utils
from telethon import functions
import asyncio
from datetime import timedelta

@loader.tds
class BlockingUser(loader.Module):
    strings = {"name": "BlockingUser"}

    @loader.command("bl", prefixes="", aliases=["bl"])
    async def bl(self, message):
        """Добавляет пользователя в черный список на указанное время (пример: bl 10 m)."""
        if message.reply_to_msg_id:
            replied_message = await message.get_reply_message()
            user_id = replied_message.sender_id
        else:
            await utils.answer(message, "Пожалуйста, ответьте на сообщение пользователя, которого нужно заблокировать.")
            return

        args = message.text.split(maxsplit=2)
        if len(args) < 3 or not args[1].isdigit() or args[2] not in ["d", "h", "m", "s"]:
            await utils.answer(message, "Укажите время блокировки в формате: {число} {d/h/m/s}.")
            return
            
        time_value = int(args[1])
        currenttime = args[2]

        if currenttime == "d":
            block_duration = timedelta(days=time_value)
        elif currenttime == "h":
            block_duration = timedelta(hours=time_value)
        elif currenttime == "m":
            block_duration = timedelta(minutes=time_value)
        elif currenttime == "s":
            block_duration = timedelta(seconds=time_value)
        else:
            await utils.answer(message, "Неверная единица времени. Используйте d, h, m или s.")
            return

        await message.client(functions.contacts.BlockRequest(user_id))
        await utils.answer(message, f"Вас заблокировали на {time_value} {currenttime}.")

        await asyncio.sleep(block_duration.total_seconds())

        await message.client(functions.contacts.UnblockRequest(user_id))
        await utils.answer(message, f"Ваша блокировка снята спустя {time_value} {currenttime}.")
