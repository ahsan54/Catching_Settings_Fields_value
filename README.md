# Catching_Settings_Fields_value Odoo-16
# Imp Point: Journal_id only exists in acc.move model not other and writing this field by inheriting acc.payment makes field non touchAble thats why i used default and not writed the field again...

#####################-----------------------> Settings Say Kisi Bi Field Ki Value Ko Assay Pkro:---->
	Logic:-
		rec_name = self.env['ir.config_parameter'].sudo().get_param('base.technical_name_here')
		
	Exampple:
			journal_id = self.env['ir.config_parameter'].sudo().get_param('base.payment_default_journal')


---------------------------------> Here i Added Field in Settings And Picked Its Value As Default Val For Other Field In Other Model:----


class AccountingSettings(models.TransientModel):
    _inherit = 'res.config.settings'

    payment_default_journal = fields.Many2one(
        'account.journal',
        string="Default Payment Journal",
        config_parameter='base.payment_default_journal',
        domain="[('type', 'in', ['bank', 'cash'])]",   # mtlb sirf yahi journals show honge click krne pr field ko 
        help="Select the default journal for payments."
    )


class AccountPaymentJournal(models.Model):                        # default sa baki fields --- wo as it is preserve rahay and last ma return krwadiya
    _inherit = 'account.payment'

    def default_get(self, fields_list):
        # Fetch default values
        defaults = super(AccountPaymentJournal, self).default_get(fields_list)

        # Get the configured default journal ID from `res.config.settings`
        journal_id = self.env['ir.config_parameter'].sudo().get_param('base.payment_default_journal')

        # Check if the journal ID exists and is valid
        if journal_id and journal_id.isdigit():
            journal = self.env['account.journal'].browse(int(journal_id))
            if journal.exists() and journal.type in ['bank', 'cash']:
                defaults['journal_id'] = journal.id

        return defaults
