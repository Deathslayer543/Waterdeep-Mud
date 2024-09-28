void do_score ( CHAR_DATA *ch, char *argument )
{
    char buf[MAX_STRING_LENGTH];
    char buf2[MAX_STRING_LENGTH];
    char racename[MAX_STRING_LENGTH];
    char class_name[MAX_STRING_LENGTH];
    char name[MAX_STRING_LENGTH];
    int time;
    BUFFER *output;
    int i;

    if ( IS_NPC ( ch ) )
        return;

    output = new_buf();

    sprintf ( buf,
              "{GName{g:{x %s%s{x\n\r", ch->pcdata->cname,
              IS_NPC ( ch ) ? ", the mobile." : ch->pcdata->title );
    add_buf ( output,buf );

    sprintf ( buf, "{GDesc{g:{x " );

    if ( !IS_NPC ( ch ) )
    {
        if ( IS_NEUTRAL ( ch ) && IS_E_NEUTRAL ( ch ) )
            strcat ( buf, "True Neutral" );
        else
        {
            if ( IS_E_LAWFUL ( ch ) )
                strcat ( buf, "Lawful " );
            else if ( IS_E_CHAOTIC ( ch ) )
                strcat ( buf, "Chaotic " );
            else
                strcat ( buf, "Neutral " );

            if ( IS_GOOD ( ch ) )
                strcat ( buf, "Good" );
            else if ( IS_EVIL ( ch ) )
                strcat ( buf, "Evil" );
            else
                strcat ( buf, "Neutral" );
        }
    }

    strcpy ( racename,race_table[ch->race].name );
    strcpy ( class_name,IS_NPC ( ch ) ? "Mobile" : class_table[ch->class_].name );

    racename[0] = toupper ( racename[0] );
    class_name[0] = toupper ( class_name[0] );

    sprintf ( buf2,
              " %s %s %s {GLevel{g: {x%d {GAge{g: {x%d\n\r",
              ch->sex == 0 ? "Sexless" : ch->sex == 1 ? "Male" : "Female",
              racename, class_name, ch->level, get_age ( ch ) );
    strcat ( buf,buf2 );
    add_buf ( output,buf );

    if ( get_trust ( ch ) != ch->level )
    {
        sprintf ( buf, "{xYou are trusted at level {B%d{x.\n\r",
                  get_trust ( ch ) );
        add_buf ( output,buf );
    }

    sprintf ( buf,
              "{GHP{g:{x%d{g/{x%d {g({c%d{g) {GMana{g:{x%d{g/{x%d {g({c%d{g) {GMove{g:{x%d{g/{x%d{x {g({c%d{g) {GSaves{g:{x%d{x\n\r",
              ch->hit,  ch->max_hit, ch->pcdata->perm_hit, ch->mana, ch->max_mana, ch->pcdata->perm_mana,
              ch->move, ch->max_move, ch->pcdata->perm_move, ch->saving_throw );
    add_buf ( output,buf );

    sprintf ( buf, "{g------------+-----------------------+---------------{x\n\r" );
    add_buf ( output,buf );

    sprintf ( buf, "{GStr{g: {x%2d{g({x%2d{g) {g| {GItems{g:{x%7d{g/{x%-7d {g| {GHitroll{g:{x%4d\n\r",
              ch->perm_stat[STAT_STR], get_curr_stat ( ch,STAT_STR ),
              ch->carry_number, can_carry_n ( ch ), GET_HITROLL ( ch ) );
    add_buf ( output,buf );

    sprintf ( buf, "{GInt{g: {x%2d{g({x%2d{g) {g| {GWeight{g:{x%6ld{g/{x%-7d {g| {GDamroll{g:{x%4d\n\r",
              ch->perm_stat[STAT_INT], get_curr_stat ( ch,STAT_INT ),
              get_carry_weight ( ch ) / 10, can_carry_w ( ch ) /10, GET_DAMROLL ( ch ) );
    add_buf ( output,buf );

    sprintf ( buf, "{GWis{g: {x%2d{g({x%2d{g) {g+----------------+------+---------------{x\n\r",
              ch->perm_stat[STAT_WIS], get_curr_stat ( ch,STAT_WIS ) );
    add_buf ( output,buf );

    sprintf ( buf, "{GDex{g: {x%2d{g({x%2d{g) {g| {GPractices{g:{x%4d {g| {GArena Wins{g:{x%6d\n\r",
              ch->perm_stat[STAT_DEX], get_curr_stat ( ch,STAT_DEX ),
              ch->practice, ch->arena_win );
    add_buf ( output,buf );

    sprintf ( buf, "{GCon{g: {x%2d{g({x%2d{g) {g| {GTrains{g:{x%7d {g| {GArena Losses{g:{x%4d\n\r",
              ch->perm_stat[STAT_CON], get_curr_stat ( ch,STAT_CON ),
              ch->train, ch->arena_loss );
    add_buf ( output,buf );

    sprintf ( buf, "{g------------+---------+------+----------------------{x\n\r" );
    add_buf ( output,buf );

    if ( !IS_NPC ( ch ) && ch->level <= LEVEL_HERO )
    {
        sprintf ( buf, "{GCoins Platinum{g:{x%6ld {g| {GExperience{x\n\r", ch->platinum );
        add_buf ( output,buf );

        sprintf ( buf, "{G      Gold{g:{x%10ld {g|  {GCurrent{g: {x%ld\n\r", ch->gold, ch->exp );
        add_buf ( output,buf );

        if ( ch->level == LEVEL_HERO )
        {
            if(IS_PK(ch)) {
                sprintf ( buf, "{G      Silver{g:{x%8ld {g|  {GPKScore{g:  {x%d\n\r", ch->silver,  get_pkrank_score(ch));
                add_buf ( output,buf );
            } else {
                sprintf ( buf, "{G      Silver{g:{x%8ld {g| {x\n\r", ch->silver );
                add_buf ( output,buf );
            }
        }
        else
        {
            sprintf ( buf, "{G      Silver{g:{x%8ld {g|  {GNeeded{g:  {x%ld\n\r",
                      ch->silver, ( ( ch->level + 1 ) * exp_per_level ( ch,ch->pcdata->points ) - ch->exp ) );
            add_buf ( output,buf );
        }
    }
    else
    {
        sprintf ( buf, "{GCoins Platinum{g:{x%6ld {g| {GHoly Light{g:{x%4s\n\r",
                  ch->platinum, IS_SET ( ch->act,PLR_HOLYLIGHT ) ?"yes":"no" );
        add_buf ( output,buf );

        sprintf ( buf, "{G      Gold{g:{x%10ld {g| {GInvisible{g:{x%5d\n\r", ch->gold, ch->invis_level );
        add_buf ( output,buf );

        sprintf ( buf,
                  "{G      Silver{g:{x%8ld {g| {GIncognito{g:{x%5d\n\r", ch->silver, ch->incog_level );
        add_buf ( output,buf );
    }

    sprintf ( buf, "{G      {GAP: {x%11d {g| \n\r", ch->pcdata->adventure_points );
    add_buf ( output,buf );

    sprintf ( buf, "{g----------------------+-----------------------------{x\n\r" );
    add_buf ( output,buf );

    for ( i = 0; i < 4; i++ )
    {

        switch ( i )
        {
        case ( AC_PIERCE ) :
            sprintf ( buf, "{GArmor Pierce{g:{x%6d [", GET_AC ( ch,AC_PIERCE ) );
            break;

        case ( AC_BASH ) :
            sprintf ( buf, "{G      Bash{g:{x%8d [", GET_AC ( ch,AC_BASH ) );
            break;

        case ( AC_SLASH ) :
            sprintf ( buf, "{G      Slash{g:{x%7d [", GET_AC ( ch,AC_PIERCE ) );
            break;

        case ( AC_EXOTIC ) :
            sprintf ( buf, "{G      Magic{g:{x%7d [", GET_AC ( ch,AC_EXOTIC ) );
            break;

        default:
            break;
        }

        add_buf ( output,buf );

        if      ( GET_AC ( ch,i ) >=  101 )
            sprintf ( buf,"{Rhopelessly vulnerable" );
        else if ( GET_AC ( ch,i ) >= 80 )
            sprintf ( buf,"{Rdefenseless" );
        else if ( GET_AC ( ch,i ) >= 60 )
            sprintf ( buf,"{Rbarely protected" );
        else if ( GET_AC ( ch,i ) >= 40 )
            sprintf ( buf,"{yslightly armored" );
        else if ( GET_AC ( ch,i ) >= 20 )
            sprintf ( buf,"{ysomewhat armored" );
        else if ( GET_AC ( ch,i ) >= 0 )
            sprintf ( buf,"{yarmored" );
        else if ( GET_AC ( ch,i ) >= -20 )
            sprintf ( buf,"{ywell-armored" );
        else if ( GET_AC ( ch,i ) >= -40 )
            sprintf ( buf,"{yvery well-armored" );
        else if ( GET_AC ( ch,i ) >= -60 )
            sprintf ( buf,"{Bheavily armored" );
        else if ( GET_AC ( ch,i ) >= -100 )
            sprintf ( buf,"{Bsuperbly armored" );
        else if ( GET_AC ( ch,i ) >= -160 )
            sprintf ( buf,"{Balmost invulnerable" );
        else if ( GET_AC ( ch,i ) >= -270 )
            sprintf ( buf,"{Wdivinely armored" );
        else if ( GET_AC ( ch,i ) >= -330 )
            sprintf ( buf,"{Yarm{wored {ylike a woo{wden sh{yed" );
        else if ( GET_AC ( ch,i ) >= -450 )
            sprintf ( buf,"{Rarm{1ored {8like a bri{1ck ho{Ruse" );
        else if ( GET_AC ( ch,i ) >= -550 )
            sprintf ( buf,"{barm{Bored {*like an ir{Bon v{bault" );
        else if ( GET_AC ( ch,i ) >= -650 )
            sprintf ( buf,"{8ar{3mor{Yed {8like an i{Yro{3n ta{8nk" );
        else if ( GET_AC ( ch,i ) >= -750 )
            sprintf ( buf,"{8ar{6mor{^ed {8like a {^fo{6rtr{8ess" );
        else if ( GET_AC ( ch,i ) >= -850 )
            sprintf ( buf,"{8ar{3mor{Yed {8like {^Fo{6rt Kno{*cks" );
        else
            sprintf ( buf,"{Rarm{1ored {8like an Ato{^mi{6c Bu{*nker" );

        strcat ( buf,"{x]" );
        sprintf ( buf2, "%-30s\n\r", buf );
        add_buf ( output,buf2 );
    }

    sprintf ( buf, "{g----------------+-------------+---------------------{x\n\r" );
    add_buf ( output,buf );

    sprintf ( buf, "{GAlignment{g:{x%5d {g| {GWimpy{g:{x%5d {g| {GQuest Points{g:{x%5d\n\r"
              , ch->alignment, ch->wimpy, ch->pcdata->questpoints );
    add_buf ( output,buf );

    sprintf ( buf, "{g----------------+-------------+---------------------{x\n\r" );
    add_buf ( output,buf );

    if ( ch->pcdata->pk_status & NPK )
    {
        sprintf ( buf, "{GPkStatus:{x NPK \n\r" );
    }

    if ( ch->pcdata->pk_status & TPK )
    {
        sprintf ( buf, "{GPkStatus:{x TPK \n\r" );
    }

    if ( ch->pcdata->pk_status & PK )
    {
        sprintf ( buf, "{GPkStatus:{x PK  \n\r" );
    }

    if ( ch->pcdata->pk_status & TNPK )
    {
        sprintf ( buf, "{GPkStatus:{x TNPK\n\r" );
    }

    if ( ch->pcdata->pk_status & HPK ) {
        sprintf ( buf, "{GPkStatus:{x HPK\n\r" );
    }
    add_buf ( output,buf );

    sprintf ( buf, "{g----------------------------------------------------{x\n\r" );
    add_buf ( output,buf );

    if ( !IS_NPC ( ch ) && ch->pcdata->condition[COND_DRUNK]   > 10 )
    {
        sprintf ( buf, "{y*HIC* yOu ArE dRuNk!!!{x\n\r" );
        add_buf ( output,buf );
    }

    if ( !IS_NPC ( ch ) && ch->pcdata->condition[COND_THIRST] ==  0 )
    {
        sprintf ( buf, "{yYour mouth has suddenly become dry.{x\n\r" );
        add_buf ( output,buf );
    }

    if ( !IS_NPC ( ch ) && ch->pcdata->condition[COND_HUNGER]   ==  0 )
    {
        sprintf ( buf, "{yThe beast within growls for food.{x\n\r" );
        add_buf ( output,buf );
    }

    switch ( ch->position )
    {
    case POS_DEAD:
        sprintf ( buf, "{RYou are DEAD!!{x\n\r" );
        add_buf ( output,buf );
        break;

    case POS_MORTAL:
        sprintf ( buf, "{RYou are mortally wounded.{x\n\r" );
        add_buf ( output,buf );
        break;

    case POS_INCAP:
        sprintf ( buf, "{RYou are incapacitated.{x\n\r" );
        add_buf ( output,buf );
        break;

    case POS_STUNNED:
        sprintf ( buf, "{RYou are stunned.{x\n\r" );
        add_buf ( output,buf );
        break;

    case POS_SLEEPING:
        sprintf ( buf, "{BYou are sleeping.{x\n\r" );
        add_buf ( output,buf );
        break;

    case POS_RESTING:
        sprintf ( buf, "{BYou are resting.{x\n\r" );
        add_buf ( output,buf );
        break;

    case POS_STANDING:
        break;

    case POS_FIGHTING:
        sprintf ( buf, "{RYou are fighting.{x\n\r" );
        add_buf ( output,buf );
        break;
    }

    if ( IS_MOUNTXED ( ch ) && ch->mountx != NULL )
    {
        sprintf ( buf, "{xYou are mounted on %s\n\r", ch->mountx->short_descr );
        add_buf ( output,buf );
    }

    if ( ch->fled > 0 )
    {
        sprintf ( buf, "{RYou have a fled penalty (%d){x\n\r", ch->fled );  // Changed this to show ticks of penalty. -Eppy
        add_buf ( output,buf );
    }

    if ( ch->relopen > 0 )    // Added for relocate penalty. -Eppy
    {
        sprintf ( buf, "{RYou have a relocate penalty (%d){x\n\r", ch->relopen );
        add_buf ( output, buf);
    }

    if ( ch->jail_timer > 0 )
    {
        sprintf ( buf, "{RYou have a jail penalty (%d){x\n\r", ch->jail_timer );
        add_buf ( output,buf );
    }

    if ( ch->corner_timer > 0 )
    {
        sprintf ( buf, "{RYou have a corner penalty (%d){x\n\r", ch->corner_timer );
        add_buf ( output,buf );
    }

    if ( !IS_NPC ( ch ) && ch->pcdata->arcane_penalty > 0 )
    {
        sprintf ( buf, "{RYou have an arcane spell penalty (%d){x\n\r", ch->pcdata->arcane_penalty );
        add_buf ( output,buf );
    }

    if ( !IS_NPC ( ch ) )
    {
        for ( i=0; i < MAX_CHANNELS; i++ )
        {
            get_channel_info ( ch, get_channel_id ( i ), name, &time, NULL );

            if ( time > 0 )
            {
                sprintf ( buf, "{RYou have a penalty of %d on the %s channel{x\n\r", time, name );
                add_buf ( output, buf );
            }
        }

        time = 0;

        get_channel_info ( ch, COMM_ID_QUIET, name, &time, NULL );

        if ( time > 0 )
        {
            sprintf ( buf, "{RYou have a penalty of %d on all channels{x\n\r", time );
            add_buf ( output, buf );
        }
    }

    if ( IS_SET ( ch->act2, PLR_WANTED ) )
    {
        sprintf ( buf, "{RYou are WANTED by the law.{x\n\r" );
        add_buf ( output,buf );
    }

    if ( IS_SET ( ch->act4, PLR_BOUNTY ) )
    {
        sprintf ( buf, "{RYou have a BOUNTY on your head.{x\n\r" );
        add_buf ( output,buf );
    }

    if ( ch->invited )
    {
        sprintf ( buf, "{RYou have been invited to join clan {x[{%s%s{x]\n\r",
                  clan_table[ch->invited].pkill_status && TNPK? "G": clan_table[ch->invited].pkill_status && NPK ? "B" : "M",
                  clan_table[ch->invited].who_name );
        add_buf ( output,buf );
    }

    page_to_char ( buf_string ( output ), ch );
    free_buf ( output );

    if ( IS_SET ( ch->comm,COMM_SHOW_AFFECTS ) )
        do_affects ( ch, ( char* ) "" );
}
