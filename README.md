OpenAPI 3.0 Specs: https://github.com/macgyver2k/bitpanda-openapi-specs

```csharp
using Bitpanda.RestClient;

using System;
using System.Linq;
using System.Net.Http;
using System.Text;
using System.Text.Json;
using System.Threading.Tasks;

namespace Bitpanda.TestClient
{
    class Program
    {
        static async Task Main( string[] args )
        {
            Console.OutputEncoding = Encoding.UTF8;

            var httpClient = new HttpClient()
            {
                DefaultRequestHeaders =
                {
                    {
                        "X-API-KEY",
                        "PUT_YOUR_KEY_HERE"
                    }
                }
            };
            
            var options = new JsonSerializerOptions() { WriteIndented = true };

            var tradesClient = new TradesClient( httpClient );
            var tradeResult = await tradesClient.GetAsync();

            var trades = tradeResult.Data
                .Select( _ => _.Attributes )
                .ToList();           

            var walletsClient = new WalletsClient( httpClient );
            var walletResult = await walletsClient.GetAsync();

            var wallets = walletResult.Data
                .Select( _ => _.Attributes )
                .Where( _ => _.Balance > 0m );
            
            foreach( var wallet in wallets )
            {
                var walletTrades = trades
                    .Where( _ => _.Cryptocoin_id == wallet.Cryptocoin_id )
                    .OrderBy( _ => _.Time.Date_iso8601 )
                    .ToList();
                
                Console.WriteLine( $"{ wallet.Cryptocoin_symbol}: {wallet.Balance}" );

                foreach( var trade in walletTrades )
                {
                    var amount = trade.Amount_cryptocoin;
                    var date = trade.Time.Date_iso8601;
                    var price = trade.Price;

                    var bruttoPrice = price * 0.98m;
                    var fee = price * 0.02m;

                    Console.WriteLine( $"--> {date:u}" );
                    Console.WriteLine( $"--> {trade.Type}" );
                    Console.WriteLine( $"--> {amount} @ {price} €" );
                    Console.WriteLine( $"--> {fee} € = {fee*amount}" );
                    Console.WriteLine( $"--> {bruttoPrice}" );
                    Console.WriteLine( $"--> { bruttoPrice * amount }" );
                }

                Console.WriteLine();
            }
        }
    }
}

```
