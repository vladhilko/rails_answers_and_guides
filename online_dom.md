1. What we are using?
      
      angular smart table
2. Adress for donwloading
      
      http://localhost:3000/api/estates/3/download.xls

3. Old example of using xml file generator
                  
                    def download
                      @estate = Estate.includes(rooms: :reports).find(params[:id])

                      # respond_to do |format|
                      #   format.xls { send_data @estate.rooms.to_csv(params[:date], @estate.full_address) }
                      # end
                      # respond_to do |format|
                      #   format.html
                      #   format.xls { render layout: false, template: "estates/paper.xlsx.axlsx", formats: [:xlsx]}
                      # end
                      # format.html

                      render layout: false, disposition: 'inline', template: "estates/download.xlsx.axlsx", formats: [:xlsx]
                      # xlsx = render_to_string layout: false, handlers: [:axlsx], formats: [:xlsx], template: "estates/paper.xlsx.axlsx"
                      # attachment = Base64.encode64(xlsx)
                      # attachments = {mime_type: Mime::XLSX, content: attachment, encoding: 'base64'}
                      # respond_to do |format| 
                      #    format.xlsx {render xlsx: 'paper',filename: "paper.xlsx.axlsx"}
                      # end
                      # respond_to do |format|
                      #   format.xls do
                      #     headers["Content-Disposition"] = "attachment;"
                      #     p = Axlsx::Package.new
                      #     wb = p.workbook
                      #     wb.add_worksheet(name: "Your worksheet name") do |sheet|
                      #       # Add your stuff
                      #     end
                      #     send_data p.to_stream.read, type: "application/xlsx", filename: "filename.xlsx"
                      #   end
                      # end
                    end
      
4. plan 
            Tariff
              
              has_many tarriff_archieves, dependent: nullify
              
              
            rails g model TariffArchive tariff:references updating_date:date old_value:decimal new_value:decimal
            
            
              set callback after update and check old and new params if they whill changed -> createt TariffArchieve object
              
              
              
           