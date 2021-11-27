Enumeración con un usuario válido del servicio RCP, por defecto dominio local WORKGROUP
`rcpclient -U "amanda" 10.10.10.103`
`rpcclient -U "" 10.10.10.52`		//Sin credenciales
`rpcclient -U "SVC_TGS%GPPStillStandingStron2K18" 10.10.10.100`		//Con credenciales

Enumeración con un usuario válido de un dominio
`rpcclient -U "amanda" -w "htb.local" 10.10.10.103`
`rpcclient -U "<usuario>%<contraseña>" 10.10.10.192 -c 'enumdomusers'`

Comandos sobre consola rcpclient
>enumeración de usuarios
>`enumdomusers`
>enumeración de grupos de usuarios
>`enumdomgroups`
>información sobre miembros de un grupo
>`querygroupmem <rid>`
>información sobre usuarios en concreto
>`queryuser <rid>`
>Información sobre grupos
>`querygroup <rid>`

Función con bash para obtener usuarios según su RID en un oneliner
>`for rid in $(rpcclient -U "<usuario>%<contraseña>" 10.10.10.100 -c "querygroupmem 0x200" | grep -oP '\[.*?\]' | grep -v "0x7" | tr -d '[]'); do echo -e "\n[*] Para el RID $rid:\n";rpcclient -U "<usuario>%<contraseña>" 10.10.10.100 -c "queryuser $rid" | grep -i "User Name"; done`

