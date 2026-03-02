import express from "express";
import cors from "cors";
import dotenv from "dotenv";
import nodemailer from "nodemailer";
import { createClient } from "@supabase/supabase-js";

dotenv.config();

const app = express();
app.use(cors());
app.use(express.json());

const supabase = createClient(
  process.env.SUPABASE_URL,
  process.env.SUPABASE_SERVICE_KEY
);

const transporter = nodemailer.createTransport({
  service: "gmail",
  auth: {
    user: process.env.EMAIL_USER,
    pass: process.env.EMAIL_PASS
  }
});

app.post("/criar-pedido", async (req, res) => {
  try {
    const { nome, email, telefone, tipo_pedido } = req.body;

    const { data: user } = await supabase
      .from("users")
      .insert([{ nome, email, telefone }])
      .select()
      .single();

    const { data: pedido } = await supabase
      .from("pedidos")
      .insert([
        {
          user_id: user.id,
          tipo_pedido,
          status: "aguardando_pagamento"
        }
      ])
      .select()
      .single();

    await transporter.sendMail({
      from: process.env.EMAIL_USER,
      to: "moitahdigital@gmail.com",
      subject: "Novo Pedido - AcolheMe",
      html: `
        <h2>Novo pedido recebido</h2>
        <p><strong>Nome:</strong> ${nome}</p>
        <p><strong>Email:</strong> ${email}</p>
        <p><strong>Telefone:</strong> ${telefone}</p>
        <p><strong>Tipo:</strong> ${tipo_pedido}</p>
        <p><strong>Status:</strong> Aguardando verificação manual no InfinityPay</p>
      `
    });

    res.json({
      success: true,
      whatsapp_link: `https://wa.me/5533999284680?text=Novo pedido AcolheMe - ${nome} - ${tipo_pedido}`
    });

  } catch (error) {
    console.error(error);
    res.status(500).json({ error: "Erro ao criar pedido" });
  }
});

app.post("/liberar-acesso", async (req, res) => {
  try {
    const { pedido_id } = req.body;

    await supabase
      .from("pedidos")
      .update({ status: "pago" })
      .eq("id", pedido_id);

    res.json({ success: true });

  } catch (error) {
    res.status(500).json({ error: "Erro ao liberar acesso" });
  }
});

app.get("/", (req, res) => {
  res.send("AcolheMe Backend Online 🚀");
});

const PORT = process.env.PORT || 3000;
app.listen(PORT, () => {
  console.log(`Servidor rodando na porta ${PORT}`);
});
