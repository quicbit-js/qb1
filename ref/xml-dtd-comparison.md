# compare with dtd:

## Example from http://www.apple.com/DTDs/PropertyList-1.0.dtd 

    <!ENTITY % plistObject "(array | data | date | dict | real | integer | string | true | false )" >
    <!ELEMENT plist %plistObject;>
    <!ATTLIST plist version CDATA "1.0" >
    
    <!-- Collections -->
    <!ELEMENT array (%plistObject;)*>
    <!ELEMENT dict (key, %plistObject;)*>
    <!ELEMENT key (#PCDATA)>
    
    <!--- Primitive types -->
    <!ELEMENT string (#PCDATA)>
    <!ELEMENT data (#PCDATA)> <!-- Contents interpreted as Base-64 encoded -->
    <!ELEMENT date (#PCDATA)> <!-- Contents should conform to a subset of ISO 8601 (in particular, YYYY '-' MM '-' DD 'T' HH ':' MM ':' SS 'Z'.  Smaller units may be omitted with a loss of precision) -->
    
    <!-- Numerical primitives -->
    <!ELEMENT true EMPTY>  <!-- Boolean constant true -->
    <!ELEMENT false EMPTY> <!-- Boolean constant false -->
    <!ELEMENT real (#PCDATA)> <!-- Contents should represent a floating point number matching ("+" | "-")? d+ ("."d*)? ("E" ("+" | "-") d+)? where d is a digit 0-9.  -->
    <!ELEMENT integer (#PCDATA)> <!-- Contents should represent a (possibly signed) integer number in base 10 -->
    
Menu definitions as xml plists

    <?xml version="1.0" encoding="UTF-8"?>
    <!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
    <plist version="1.0">
    <array>
    	<dict>
    		<key>Title</key>
    		<string>Knowledge</string>
    		<key>KeyEquivalent</key>
    		<string>k</string>
    		<key>ShiftModifier</key>
    		<false/>
    		<key>OptionModifier</key>
    		<false/>
    		<key>AngbandCommand</key>
    		<string>~</string>
    	</dict>
    	<dict>
    		<key>Title</key>
    		<string>List Monsters</string>
    		<key>KeyEquivalent</key>
    		<string>l</string>
    		<key>ShiftModifier</key>
    		<false/>
    		<key>OptionModifier</key>
    		<false/>
    		<key>AngbandCommand</key>
    		<string>[</string>
    	</dict>
    	<dict>
    		<key>Title</key>
    		<string>List Items</string>
    		<key>KeyEquivalent</key>
    		<string>l</string>
    		<key>ShiftModifier</key>
    		<true/>
    		<key>OptionModifier</key>
    		<false/>
    		<key>AngbandCommand</key>
    		<string>]</string>
    	</dict>
    	<dict>
    		<key>Title</key>
    		<string>Inscribe</string>
    		<key>KeyEquivalent</key>
    		<string>i</string>
    		<key>ShiftModifier</key>
    		<false/>
    		<key>OptionModifier</key>
    		<false/>
    		<key>AngbandCommand</key>
    		<string>{</string>
    	</dict>
    	<dict>
    		<key>Title</key>
    		<string>Uninscribe</string>
    		<key>KeyEquivalent</key>
    		<string>i</string>
    		<key>ShiftModifier</key>
    		<true/>
    		<key>OptionModifier</key>
    		<false/>
    		<key>AngbandCommand</key>
    		<string>}</string>
    	</dict>
    </array>
    </plist>

Definition as qb1

    {
        $format: 'qb1'
        $type: [ {} ]
        $value: [
            {
                Title: List Monsters
                KeyEquivalent: l
                ShiftModifier: false
                OptionModifier: false
                AngbandCommand: [
            }
            {
                Title: List Items
                KeyEquivalent: l
                ShiftModifier: true
                OptionModifier: false
                AngbandCommand: ]
            }
            {
                Title: Knowledge
                KeyEquivalent: k
                ShiftModifier: false
                OptionModifier: false
                AngbandCommand: ~
            }
            {
                Title: Inscribe
                KeyEquivalent: i
                ShiftModifier: false
                OptionModifier: false
                AngbandCommand: {
            }
            {
                Title: Uninscribe
                KeyEquivalent: i
                ShiftModifier: true
                OptionModifier: false
                AngbandCommand: }
            }
        ]
    }            

While we are talking about data representation, let me recommend more concise identifiers:

    {
        $format: 'qb1'
        $type: [ {} ]
        $value: [
            {
                title: List Monsters
                key: l
                cmd: [
            }
            {
                title: List Items
                key: shift-l                    ^ perhaps use a mini-syntax instead of the extra properties?
                cmd: ]
            }
            {
                title: Knowledge
                key: k
                cmd: ~
            }
            {
                title: Inscribe
                key: i
                cmd: {
            }
            {
                title: Uninscribe
                key: i
                cmd: }
            }
        ]
    }
         
There are good reasons for avoiding mixed case public identifiers - identity-in-case can cause problems
when it shows up in URLs, compound identifiers and file paths.  Best to avoid:
    
    TitleAndKeyEquivalent
    
    http://local/menu_service/KeyEquivalent=l&ShiftModifier=false&OptionModifer=false 
    
    /files/menu/index/KeyEquivalent-AngbandCommand

By using:

    title_and_key
    
    http://local/menuservice?title=List+Monsters&key=shift-l

    /files/menu/index/key-command

    
In qb1, we can write information as a table by declaring the $type structure.  Here is the same data
in a more concise form:

    {
        $format: 'qb1'
        $type: [ { shift: boo, option: boo, key: str, cmd: str, title: str } ]
        $value: [
            ^ shift, option, key, cmd, title             - this header is just a comment, for clarity
            [ false,   false,    l,  ']',  List Monster ]  
            [ true,    false,    l,  '[',  List Items ]    
            [ false,   false,    k,  '~',  Knowledge ]     
            [ false,   false,    i,  '{',  Inscribe ]      
            [ false,   false,    I,  '}',  Uninscribe ]    
        ]
    }     
    
Or squish it down to fewer bytes with smaller margins and 1's and 0's for booleans and qb1 "tinyname" type identifiers:

    {
      $format:'qb1'
      $type:[{shift:b,option:b,key:s,cmd:s,title:s}]
      $value: [
        [0,0,l,']',ListMonster]  
        [1,0,l,'[',ListItems]    
        [0,0,k,'~',Knowledge]     
        [0,0,i,'{',Inscribe]      
        [0,0,I,'}',Uninscribe]    
      ]
    }
    
Or without whitespace for 173 bytes small-

    {$format:'qb1',$type:[{shift:b,option:b,key:s,cmd:s,title:s}],$value:[[0,0,l,']',ListMonster],[1,0,l,'[',ListItems],
    [0,0,k,'~',Knowledge],[0,0,i,'{',Inscribe],[0,0,I,'}',Uninscribe]]}

 Declaring the type structure in qb1 gives flexibility in representing the data because it allows
 qb1 to make sensible interpretations; coerce array-of-array into array-of-object, coerce one/zero into
 true/false etc...