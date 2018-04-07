# CYML

CYML is a simple language for building single page interactive fiction. Here is an example.

~~~
<!room := "entrance">
<!gameOver := false>
<!inventory := []>
<?gameOver|
  <:true|
    You leave the house, your mind blank and the photograph in your pocket.>
  <:false|
    <?room|
      <:"entrance"|
        <!keyInEntrance := true>
        You enter the house. 
        <?: keyInEntrance|A <i|key> is on the sidetable. 
          <@ inventory.add("key"); keyInEntrance = false|Take the key.>>

        <@ room="dining"|Move to the dining room.>>
        
        <?: inventory.includes("photograph")|
          <@ gameOver=true|Leave.>>>
      <:"dining"|
        <!safeOpen := false>
        The dining room is large and empty. A safe is on the table.

        <?safeOpen|
          <!photographInSafe := true>
          <:true|The safe is open.
            <?photographInSafe|
              <:true|You see a photograph in the open safe.
                <@ inventory.add("photograph"); photographInSafe=false|
                  Take it.>>
              <:false|The safe is empty>>
          <:false|The safe is closed. A key could open it.
            <?: inventory.includes("key")|<@ safeOpen = true|Open it.>>>

        <@ room="entrance"|Return to the entrance.>>>>
~~~
