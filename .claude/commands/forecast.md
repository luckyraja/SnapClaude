# forecast Command

Use Websearch to look up upcoming Marvel Snap cards. If the user provides card name as a parameter, provide specific information on that card. 

## Card to Evaluate:
{CARD_NAME}

## forecast with no card name
Use websearch to look up all official and datamined cards coming in the next 1-2 months in marvel snap. 

Return the cards in the following format
1 Card Name
2. Release Date
3. Official release or Datamined release
4. Predict what card archetypes the card will work with

## forecast with provided card name
Use websearch to look up a specific unreleased marvel snap card

Return the following info
1. Card name
2. Card ability description
4. Predict what card archetyupes the card will work with. Reference the users card collection in the csv starting with marvel_snap_collection. Offer predicted card synergies. 