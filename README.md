#!/bin/bash

CONFIG_DIR="/etc/opt/remi/php83/php-fpm.d"

if [ ! -d "$CONFIG_DIR" ]; then
  echo "Diretório de configuração não encontrado: $CONFIG_DIR"
  exit 1
fi

CONF_FILES=$(find "$CONFIG_DIR" -maxdepth 1 -type f -name "*.conf")

if [ -z "$CONF_FILES" ]; then
  echo "Nenhum arquivo .conf encontrado em $CONFIG_DIR"
  exit 1
fi

for FILE in $CONF_FILES; do
  MAX_CHILDREN=$(grep -E "^pm.max_children\s*=" "$FILE" | awk -F '=' '{print $2}' | tr -d ' ')
  POOL_NAME=$(basename "$FILE")
  if [ -n "$MAX_CHILDREN" ]; then
    echo "Pool: $POOL_NAME"
    echo "pm.max_children = $MAX_CHILDREN"
    echo "----------------------------"
  else
    echo "Pool: $POOL_NAME"
    echo "pm.max_children não configurado."
    echo "----------------------------"
  fi
done


--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------



Como usar:
Crie o arquivo check_all_pools_max_children.sh:

bash
Copiar
Editar
nano check_all_pools_max_children.sh
Cole o script acima, salve (Ctrl + O) e saia (Ctrl + X).

Dê permissão de execução:

bash
Copiar
Editar
chmod +x check_all_pools_max_children.sh
Rode:

bash
Copiar
Editar
./check_all_pools_max_children.sh


















    
