# CQRS

## Intro

1. Api dispatches a COMMAND
2. This command is handled by a COMMAND HANDLER
3. The command handler dispatches an EVENT on the corresponding AGGREGATE
4. A PROCESS MANAGER listens to the event and write necessary data to the DB
5. The aggregate also listens to the event to write to the event stream 

## Schema

