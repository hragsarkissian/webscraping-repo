# webscraping-repo

This is a demonstration of basic web scraping that is implemented on server-side and saved into the database for every link posted on the entire platform.

//Web Scraping in the class of web api controller.

[Route, HttpPost]

        public HttpResponseMessage Post(LinksAddRequest model)
        {
        
            var url = model.Url;
            var webGet = new HtmlAgilityPack.HtmlWeb();
            HtmlAgilityPack.HtmlDocument doc = webGet.Load(url);
            HtmlWeb web = new HtmlWeb(url);
            string titleNodes = doc.DocumentNode.SelectSingleNode("//head/title").InnerText;
            var descriptionNodes = doc.DocumentNode.SelectSingleNode("//head/meta[@name='description']").Attributes["content"].Value;
            var imageNodes = doc.DocumentNode.SelectSingleNode("//head/meta[@property='og:image']").Attributes["content"].Value;
            
            model.Title = titleNodes;
            model.Description = descriptionNodes;
            model.Image = imageNodes;
            //  model.Description = descriptionNodes;

            try
            {
                if (ModelState.IsValid)
                {
                    model.ModifiedBy = "b69fcb16-3da2-450b-b4ca-f692c6184ffe";
                    int id = _service.Insert(model);
                    ItemResponse<int> resp = new ItemResponse<int>();
                    resp.Item = id;
                    return Request.CreateResponse(HttpStatusCode.OK, resp);
                }
                else
                {
                    return Request.CreateErrorResponse(HttpStatusCode.BadRequest, ModelState);
                }
            }
            catch (Exception ex)
            {
                return Request.CreateErrorResponse(HttpStatusCode.BadRequest, ex.Message);
            }
        }
        
    // The Service class that inserts the data into the sql server database
    
    public class LinksService : BaseService, ILinksService
    {
        public int Insert(LinksAddRequest model)
        {
            int id = 0;
            this.DataProvider.ExecuteNonQuery(
                "Project_Links_Insert",
                inputParamMapper: delegate (SqlParameterCollection paramCol)
                {
                    SqlParameter param = new SqlParameter();
                    param.ParameterName = "@Id";
                    param.SqlDbType = System.Data.SqlDbType.Int;
                    param.Direction = System.Data.ParameterDirection.Output;
                    paramCol.Add(param);

                    paramCol.AddWithValue("Title", model.Title);
                    paramCol.AddWithValue("Description", model.Description);
                    paramCol.AddWithValue("Url", model.Url);
                    paramCol.AddWithValue("Image", model.Image);
                    paramCol.AddWithValue("ModifiedBy", model.ModifiedBy);
                },
                returnParameters: delegate (SqlParameterCollection paramCol)
                {
                    id = (int)paramCol["@Id"].Value;
                }
            );
            return id;
        }
