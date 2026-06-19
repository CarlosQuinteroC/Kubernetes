# Kubernetes CKA Study Notes

This repository is a personal study notebook for preparing for the **Certified Kubernetes Administrator (CKA)** exam.

The goal is not only to collect commands, but to explain **why each command is used, when it matters, and in what order it should be executed**. Many Kubernetes guides show the final command sequence, but they often skip the reasoning behind each step. This repo is meant to close that gap.

## What this repo is for

- Practicing Kubernetes administration tasks.
- Understanding the reasoning behind setup and troubleshooting commands.
- Building a repeatable study path for the CKA exam.
- Keeping explanations close to the commands so they are easier to review later.

## Repository structure

| Folder | Purpose |
|--------|---------|
| [`ClusterSetUp`](./ClusterSetUp) | Notes for creating a Kubernetes cluster, including control plane setup and worker node joining. |

## Current notes

| Topic | File |
|-------|------|
| Control plane setup | [`ClusterSetUp/ControlPlaneSetup.md`](./ClusterSetUp/ControlPlaneSetup.md) |
| Worker node setup | [`ClusterSetUp/WorkerSetup.md`](./ClusterSetUp/WorkerSetup.md) |

## Why `ClusterSetUp` exists

This first folder was created because many cluster setup guides explain **what to run**, but not:

- why one command is needed before another;
- why one runtime, package, or configuration is chosen over another;
- which commands run on the control plane and which run on worker nodes;
- which values are examples and which values must match your own environment.

The notes are written as a learning path, not just a copy-paste script.

## How to use these notes

1. Read the explanation before running the command.
2. Pay attention to which node the command belongs to: control plane or worker.
3. Replace sample IPs, tokens, hostnames, and versions with values from your own lab.
4. After each section, verify the result with `kubectl`, `systemctl`, or the expected output shown in the notes.

## Important note

These notes are for learning and practice. Kubernetes versions, package repositories, and exam expectations can change, so always compare the commands with the official Kubernetes documentation when preparing for the real CKA exam.

---

# Apuntes de estudio para Kubernetes CKA

Este repositorio es una libreta personal de estudio para prepararse para el examen **Certified Kubernetes Administrator (CKA)**.

El objetivo no es solo juntar comandos, sino explicar **por qué se usa cada comando, cuándo importa y en qué orden conviene ejecutarlo**. Muchas guías de Kubernetes muestran la secuencia final, pero se saltean el razonamiento detrás de cada paso. Este repo busca cubrir ese vacío.

## Para qué sirve este repo

- Practicar tareas de administración de Kubernetes.
- Entender el razonamiento detrás de comandos de instalación, configuración y troubleshooting.
- Construir una ruta de estudio repetible para el CKA.
- Mantener las explicaciones cerca de los comandos para repasarlas con menos fricción.

## Estructura del repositorio

| Carpeta | Propósito |
|---------|-----------|
| [`ClusterSetUp`](./ClusterSetUp) | Apuntes para crear un cluster de Kubernetes, incluyendo la configuración del control plane y la unión de worker nodes. |

## Apuntes actuales

| Tema | Archivo |
|------|---------|
| Configuración del control plane | [`ClusterSetUp/ControlPlaneSetup.md`](./ClusterSetUp/ControlPlaneSetup.md) |
| Configuración de worker node | [`ClusterSetUp/WorkerSetup.md`](./ClusterSetUp/WorkerSetup.md) |

## Por qué existe `ClusterSetUp`

Esta primera carpeta nace porque muchas guías de creación de clusters explican **qué ejecutar**, pero no explican bien:

- por qué un comando tiene que ir antes que otro;
- por qué se elige un runtime, paquete o archivo de configuración;
- qué comandos se ejecutan en el control plane y cuáles en los worker nodes;
- qué valores son ejemplos y cuáles dependen del laboratorio propio.

Los apuntes están pensados como una ruta de aprendizaje, no como un script para copiar y pegar sin entender.

## Cómo usar estos apuntes

1. Leé la explicación antes de ejecutar el comando.
2. Prestá atención a qué nodo corresponde cada comando: control plane o worker.
3. Reemplazá IPs, tokens, hostnames y versiones de ejemplo por los valores de tu propio laboratorio.
4. Después de cada sección, verificá el resultado con `kubectl`, `systemctl` o la salida esperada que aparece en los apuntes.

## Nota importante

Estos apuntes son para aprendizaje y práctica. Las versiones de Kubernetes, los repositorios de paquetes y los objetivos del examen pueden cambiar, así que conviene comparar siempre los comandos con la documentación oficial de Kubernetes al prepararse para el CKA real.
