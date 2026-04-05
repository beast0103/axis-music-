# axis-music-require('dotenv').config();

const { Client, GatewayIntentBits } = require('discord.js');
const { joinVoiceChannel, createAudioPlayer, createAudioResource } = require('@discordjs/voice');
const play = require('play-dl');

const client = new Client({
  intents: [
    GatewayIntentBits.Guilds,
    GatewayIntentBits.GuildMessages,
    GatewayIntentBits.MessageContent,
    GatewayIntentBits.GuildVoiceStates
  ]
});

const queue = new Map();

client.once('ready', () => {
  console.log('🔥 AXIS BOT ONLINE');
});

client.on('messageCreate', async (message) => {
  if (!message.content.startsWith('!')) return;

  const args = message.content.split(' ');
  const cmd = args[0];

  // PLAY
  if (cmd === '!play') {
    const vc = message.member.voice.channel;
    if (!vc) return message.reply('Join VC first!');

    const query = args.slice(1).join(' ');
    const result = await play.search(query, { limit: 1 });

    const song = {
      title: result[0].title,
      url: result[0].url
    };

    let serverQueue = queue.get(message.guild.id);

    if (!serverQueue) {
      const connection = joinVoiceChannel({
        channelId: vc.id,
        guildId: message.guild.id,
        adapterCreator: message.guild.voiceAdapterCreator
      });

      const player = createAudioPlayer();

      serverQueue = {
        connection,
        player,
        songs: []
      };

      queue.set(message.guild.id, serverQueue);
      connection.subscribe(player);
    }

    serverQueue.songs.push(song);

    const stream = await play.stream(song.url);
    const resource = createAudioResource(stream.stream, { inputType: stream.type });

    serverQueue.player.play(resource);

    message.reply(`🎶 Playing: ${song.title}`);
  }
});

client.login(process.env.TOKEN);
