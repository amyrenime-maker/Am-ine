// ===== IMPORTS =====
const { Client, GatewayIntentBits, ActionRowBuilder, StringSelectMenuBuilder, EmbedBuilder } = require("discord.js");

// ===== CLIENT SETUP =====
const client = new Client({
    intents: [
        GatewayIntentBits.Guilds,
        GatewayIntentBits.GuildMessages,
        GatewayIntentBits.MessageContent,
        GatewayIntentBits.DirectMessages
    ]
});

// ===== FUNCTIONS =====
function generateCard(prefix, length) {
    let card = prefix;
    while (card.length < length) {
        card += Math.floor(Math.random() * 10);
    }
    return card;
}

function randomCVV(type = "normal") {
    return type === "amex"
        ? Math.floor(1000 + Math.random() * 9000).toString()
        : Math.floor(100 + Math.random() * 900).toString();
}

function randomExpiry() {
    const month = String(Math.floor(Math.random() * 12) + 1).padStart(2, "0");
    const year = String(Math.floor(Math.random() * 6) + 26);
    return `${month}/${year}`;
}

// ===== READY =====
client.once("ready", () => {
    console.log(`✅ Logged in as ${client.user.tag}`);
});

// ===== PANEL COMMAND =====
client.on("messageCreate", async (message) => {
    if (message.content === "!panel") {
        const embed = new EmbedBuilder()
            .setTitle("💳 Card Generator Panel")
            .setDescription(
                "Select a card type from the list below\n\n" +
                "• Card details will be sent to your DMs\n" +
                "• CVV & Expiry are random\n\n" +
                "⚠️ Demo / Simulation only"
            )
            .setFooter({ text: "Developed by YourName" })
            .setColor("#2b2d31");

        const menu = new StringSelectMenuBuilder()
            .setCustomId("card_select")
            .setPlaceholder("Select card type...")
            .addOptions([
                { label: "Visa", value: "visa", emoji: "💳" },
                { label: "Mastercard", value: "mastercard", emoji: "💳" },
                { label: "American Express", value: "amex", emoji: "💳" },
                { label: "Discover", value: "discover", emoji: "💳" }
            ]);

        const row = new ActionRowBuilder().addComponents(menu);
        await message.channel.send({ embeds: [embed], components: [row] });
    }
});

// ===== SELECT HANDLER =====
client.on("interactionCreate", async (interaction) => {
    if (!interaction.isStringSelectMenu()) return;
    if (interaction.customId !== "card_select") return;

    let card;
    switch(interaction.values[0]) {
        case "visa":
            card = { prefix: "4", length: 16, type: "VISA" };
            break;
        case "mastercard":
            card = { prefix: "5", length: 16, type: "MASTERCARD" };
            break;
        case "amex":
            card = { prefix: "3", length: 15, type: "AMEX" };
            break;
        case "discover":
            card = { prefix: "6", length: 16, type: "DISCOVER" };
            break;
    }

    const cardNumber = generateCard(card.prefix, card.length);
    const exp = randomExpiry();
    const cvv = card.type === "AMEX" ? randomCVV("amex") : randomCVV();

    const embed = new EmbedBuilder()
        .setTitle(`💳 ${card.type} Generated`)
        .setDescription(
            `**Number:** ${cardNumber}\n` +
            `**Expiry:** ${exp}\n` +
            `**CVV:** ${cvv}\n\n` +
            "⚠️ Demo / Simulation only"
        )
        .setColor("#f1c40f");

    try {
        await interaction.user.send({ embeds: [embed] });
        await interaction.reply({ content: "📩 Sent to your DMs", ephemeral: true });
    } catch {
        await interaction.reply({ content: "❌ Please open your DMs first.", ephemeral: true });
    }
});

// ===== LOGIN USING GITHUB SECRET =====
// ⚠️ ضع اسم السر في GitHub Secrets مثل: DISCORD_TOKEN
client.login(process.env.DISCORD_TOKEN);
