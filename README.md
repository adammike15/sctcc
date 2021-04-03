using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;

using Newtonsoft.Json;
using Amazon.DynamoDBv2;
using Amazon.DynamoDBv2.DataModel;
using Amazon.DynamoDBv2.Model;
using Amazon.DynamoDBv2.DocumentModel;
using Amazon.Lambda.APIGatewayEvents;

using Amazon.Lambda.Core;

// Assembly attribute to enable the Lambda function's JSON input to be converted into a .NET class.
[assembly: LambdaSerializer(typeof(Amazon.Lambda.Serialization.Json.JsonSerializer))]

namespace assignment7GetStats
{
    public class StatisticItem
    {
        public string category;
        public double count;
        public double totalRating;
        public double averageRating;
    }
    public class Function
    {
        private static AmazonDynamoDBClient client = new AmazonDynamoDBClient();
        private string tableName = "RatingsByType";

        public async Task<Dictionary<string,double>> FunctionHandler(APIGatewayProxyRequest input, ILambdaContext context)
        {
            //create variable to hold incoming object
            string itemCategory = "";

            //create a dictionary
            Dictionary<string, string> dict = (Dictionary<string, string>)input.QueryStringParameters;

            // itemId in this context is the submitted query string parameter
            dict.TryGetValue("category", out itemCategory);

            //Query the Dynamo DB Table
            GetItemResponse res = await client.GetItemAsync(tableName, new Dictionary<string, AttributeValue>
            {
                {"category", new AttributeValue {S = itemCategory.ToUpper()} }
            }

            );

            //convert data from the table into a document
            Document myDoc = Document.FromAttributeMap(res.Item);

            //convert document to an oject
            StatisticItem myStat = JsonConvert.DeserializeObject<StatisticItem>(myDoc.ToJson());

            //New code
            myStat.averageRating = Math.Round(myStat.totalRating / myStat.count, 1);


            //create dictionary to prune off un wanted data
            Dictionary<string, double> statDict = new Dictionary<string, double>();
            statDict.Add("count", myStat.count);
            statDict.Add("averageRating", myStat.averageRating);

            return statDict;
        }
    }
}
