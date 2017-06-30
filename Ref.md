** Backtesting is a term used in oceanography, meteorology and the financial industry to refer to testing a predictive model using existing historic data.
Backtesting is a kind of retrodiction, and a special type of cross-validation applied to time series data.


** Phase One

    def initialize(context):
        context.aapl = sid(24)
        context.spy = sid(8554)
        context.message = 'Market started: '
        context.security_list = [sid(24), sid(8554), sid(5061)]
    
    def handle_data(context, data):
        if data.can_trade(context.aapl):
            order_target_percent(context.aapl, 0.60)
        if data.can_trade(context.spy):
            order_target_percent(context.spy, -0.40)
        hist = data.history(context.security_list, 'volume', 10, '1m').mean()
        print hist.mean()   
        
        hist2 = data.history([sid(24), sid(8554), sid(5061)], ['low', 'high'], 5, '1m')
        means = hist2.mean()
        print ('...', means.mean())
        mean_lows = means['low']
        mean_highs = means['high']
        print (mean_lows, mean_highs)
            
    def before_trading_start(context, data):
        print context.message 
        print context.aapl
        print data.current([sid(24), sid(8554)], ['price', 'open', 'high', 'low', 'close', 'volume'])

        
** Phase Two

    def initialize(context):
        context.spy = sid(8554)
    
        schedule_function(open_positions, date_rules.week_start(), time_rules.market_open())
        schedule_function(close_positions, date_rules.week_end(), time_rules.market_close(minutes=30))
    
    def open_positions(context, data):
        order_target_percent(context.spy, 0.10)
    
    def close_positions(context, data):
        order_target_percent(context.spy, 0)


** Phase Three

    def initialize(context):
        context.aapl = sid(24)
        context.spy = sid(8554)
    
        schedule_function(rebalance, date_rules.every_day(), time_rules.market_open())
        schedule_function(record_vars, date_rules.every_day(), time_rules.market_close())
    
    def rebalance(context, data):
        order_target_percent(context.aapl, 0.50)
        order_target_percent(context.spy, -0.50)
    
    def record_vars(context, data):
    
        long_count = 0
        short_count = 0
    
        for position in context.portfolio.positions.itervalues():
            if position.amount > 0:
                long_count += 1
            if position.amount < 0:
                short_count += 1
    
        # Plot the counts
        record(num_long=long_count, num_short=short_count)
