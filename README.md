using cAlgo.API;
using cAlgo.API.Internals;
using cAlgo.API.Indicators;

namespace cAlgo
{
    [Robot(TimeZone = TimeZones.UTC, AccessRights = AccessRights.None)]
    public class TrendFollowingBot : Robot
    {
        [Parameter("Moving Average Period", DefaultValue = 50)]
        public int MaPeriod { get; set; }

   [Parameter("RSI Period", DefaultValue = 14)]
        public int RsiPeriod { get; set; }

  [Parameter("RSI Overbought Level", DefaultValue = 70)]
        public double RsiOverbought { get; set; }

  [Parameter("RSI Oversold Level", DefaultValue = 30)]
        public double RsiOversold { get; set; }

  [Parameter("Trade Volume", DefaultValue = 0.1, MinValue = 0.01, MaxValue = 100)]
        public double TradeVolume { get; set; }

 [Parameter("Take Profit (pips)", DefaultValue = 20)]
        public int TakeProfitPips { get; set; }

  [Parameter("Stop Loss (pips)", DefaultValue = 20)]
        public int StopLossPips { get; set; }

  private MovingAverage _movingAverage;
        private RelativeStrengthIndex _rsi;

  protected override void OnStart()
        {
            // Initialize indicators
            _movingAverage = Indicators.MovingAverage(Bars.ClosePrices, MaPeriod, MovingAverageType.Simple);
            _rsi = Indicators.RelativeStrengthIndex(Bars.ClosePrices, RsiPeriod);
        }

  protected override void OnBar()
        {
            // Check if there are open positions with the same label
            if (Positions.Find("TrendFollowingBot", Symbol) != null)
                return;

   // Get current and previous values of moving average and RSI
            double maCurrent = _movingAverage.Result.Last(1);
            double maPrevious = _movingAverage.Result.Last(2);
            double rsiCurrent = _rsi.Result.Last(1);
            double currentPrice = Bars.ClosePrices.Last(1);

  try
            {
                // Calculate take profit and stop loss prices
                double takeProfitPrice = 0;
                double stopLossPrice = 0;

  if (currentPrice > maCurrent && currentPrice > maPrevious)
                {
                    if (rsiCurrent < RsiOversold)
                    {
                        // Buy condition
                        takeProfitPrice = Symbol.Bid + TakeProfitPips * Symbol.PipSize;
                        stopLossPrice = Symbol.Bid - StopLossPips * Symbol.PipSize;
                        ExecuteMarketOrder(TradeType.Buy, Symbol, TradeVolume, "TrendFollowingBot", stopLossPrice, takeProfitPrice);
                    }
                }
                else if (currentPrice < maCurrent && currentPrice < maPrevious)
                {
                    if (rsiCurrent > RsiOverbought)
                    {
                        // Sell condition
                        takeProfitPrice = Symbol.Ask - TakeProfitPips * Symbol.PipSize;
                        stopLossPrice = Symbol.Ask + StopLossPips * Symbol.PipSize;
                        ExecuteMarketOrder(TradeType.Sell, Symbol, TradeVolume, "TrendFollowingBot", stopLossPrice, takeProfitPrice);
                    }
                }
            }
            catch (Exception ex)
            {
                // Print any errors that occur during order execution
                Print("Error executing order: " + ex.Message);
            }
        }
    }
}
